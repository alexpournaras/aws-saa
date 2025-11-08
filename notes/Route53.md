DNS
Domain name system which translates the human friendly hostnames into machine IP addresses

Domain Registar: amazon route 53, godaddy, papaki
top level domain (tld) : .com .gr .org .gov
second level domain: amazon.com google.com

subdomain: [api].amazon.com

local dns server -> asking for example.com -> root dns server (managed by ICANN) -> tld dns server (.com) (managed by IANA) -> SLD DNS Server (example.com) (managed by domain registar like amazon registar etc etc.)

--

amazon route 53

scalable fully managed and authoritative DNS
    - authoritative = you can update the DNS records

route 53 is also domain registar
100% availability sla
53 is reference to traditional dns port

the records have:
* domain/subdomain
* record type
* value
* routing policy (how route 53 respond to queries)
* ttl


record types that maps a hostname to:
A => ipv4
AAAA => ipv6
CNAME => to another hostname (cant create cname record for top node of a dns namespace zone apex)
NS => name servers for the hosted zone (control how traffic is routed for a domain)

hosted zones:
* public: application1.mypublicdomain.com
* private: application1.company.internal
0.5$ per month per hostez zone

to register domains at least 12$ per year

ttl time to live
basically cache the reponse of the dns server in order to avoid quering again and again.

high ttl (example 24 hours)
* less traffic on route 53 (less cost)
* possibly outdated records

low ttl (60 seconds)
* more traffic to route 53 ($$$)
* records are outdated for less time
* easy to change records

its mandatory for each dns record (except alias)

```bash
sudo yum install -y bind-utils
dig api.example.com
```

```bash
winget install ISC.Bind
dig api.example.com
```

--

CNAME:
* points a hostname to ANY OTHER hostname
app.mydomain.com => blabla.anything.com
* ONLY FOR NON ROOT DOMAIN like something.mydomain.com

Alias
* points a hostname to an AWS RESOURCE
app.mydomain.com => blabla.anything.com
* works for ROOT DOMAIN AND NON ROOT DOMAIN
(like mydomain.com and something.mydomain.com)
* alias is free and has native health checks
* its an extension to DNS functionality
* automatically recognisze changes in the resource's ip addresses
* its always a type A/AAAA for aws resources
* cant set ttl. aws does it automatically

alias records targets:
* elastic load balancers
* cloudfront distributions
* api gateway
* elastic beanstack environments
* s3 websites
* vps interface endpoints
* global accelerator
* route 53 record in the same hosted zone
*** cannot set alias in EC2 DNS NAME!!!!! (important)

domain apex = [example.com]

routing policies - simple

* route traffic to a single resource
* can specify multiple values in the same record
* if multiple values are returned, a random one is chosen by the CLIENT
* when alias enabled, specify only one aws resource
* cant be associated with health checks
( could be just a simple A record )

--

routing policies - weighted

* control & of the request that go to each resource
* assign a relative weight to each record
* the percentage is the weight of this record / against the total weight of all records
* weights dont need to sum up to 100
* dns records must have the same name and type
* can be assosicated with health checks
* use cases: loadbalancing between regions, testing new application versions
* assign weight 0 to a record to stop sending traffic
* if all records wave 0, they will all equally be returned
* It's a common use case to send part of traffic (like 5%) to a new version of your application to test updates.

--

routing policies - latency

* redirect to the resource that has the least leatency close to us
* super helpful when latency is priority
* latency is based on traffic between users and aws regions
* can be associated with health checks (has a failover capability)
* latency = response time

tip: when changing my ip with vpn, all ttl are cleared so we can get another response imedeatelly

--

route 53 health checks

* http health checks are only for public resources
health check => automated dns failover
- monitoring an endpoint (application, server, other aws resource)
- monitor other health checks (calculated health checks)
- monitor cloudwatch alarms. (full control in throttles of dynamodb, alarms on rds, custom metrics) [!!! helpful for private resources]
* they are integrated with cloudwatch metrics

## monitor an endpoint
* about 15 global helath checkers will check the endpoint
* failure threshhold (number of times to flagged as failed) (default = 3)
* intervals 30 sec (can set to 10 sec - higher cost)
* support http, https and tcp
* if > 18% report helaty endpoint, route 53 will consider it healthy.
* choose which locations route 53 to use.
* checks are passed with 2xx and 3xx status codes
* can be set to check texts to the first 5120 bytes of the response
* configure router/firewall to allow incomng requests from route 53 health checkers (its a specific ip range.)

## calculated health checks
* combine results of multiple health checks to a single one. (its like the parent and can monitor other child health checks)
* monitor up to 256 child health checks
* can be: 
    - at least X of Y health checks are healthy,
    - ALL health checks are healthy (AND),
    - one or more health checks are healthy (OR)
* specify how many health checks need to pass to make parent pass
* usage: perform maintance to website without causing all health checks to fail

## private hosted zone
* health checks are outside of the vpc so they cannot access privatte endpoints
* but we can create cloudwatch metrics and assiciate cloudwatch alarms, then the health check can check this alarm.

--

routing policies - failover 

* basically they work with health checks,
* one record is primary, another is secondary
* based on the health check the traffic will go to the corresponding record
* You can create only one failover record of each type.

--

routing policies - geolocation

* the routing is based on the user location
* specify continent, country, us state
* should create a 'Default' record when there is no match
* use cases: website localization, retrict content distribution, load balancing etc.
* can be associated with health checks

-- 

routing policies - geoproximity

* based on geographic location of users and resources
* ability to shift more traffic to resources based on the defined 'bias' and location
* change size of geographic region with 'bias' value. to 1 to 99 to expand, -1 to -99 to shrink

resources can be:
* aws resources( aws region)
* non-aws resources (specify latitude and logitude)
* use feature traffic flow

--

routing policies - ip based routing

* routing is based on clients ip address
* you provide a list of CIDRs for your clients (ip ranges) and map them to locations
* use cases: optimize performance, reduce network costs, route user from particular ISP to specific endpoint

example:
Route 53 -> CIDR Collection

location 1 -> 203.0.113.0/24
location 2 -> 200.5.4.0/24

records:
example.com -> 1.2.3.4 -> location 1
example.com -> 5.6.7.8 -> location 2

--

routing policies - multivalue

* route triffic to multiple resources
* route 53 will also return multiple values/resources
* can be associated with health checks (and return only values for healthy resources)
* up to 8 healthy resoruces are retured for each multivalue query
* its not substitute for ELB, its like load balacing but in the client side.
* its different with the simple because thats only returning healthy. also simple doesnt support health checks.

--

domain registar vs dns service

* the domain registars usually provide us with a dns service to manage our dns records
* but we can use another dns service to manage our dns records
* example we buy a domain from a 3rd pary website (godaddy) but we use route 53 to manage our dns records

to do that: we create a hosted zone in route 53
and then update the nameserver records on 3rd party website with the ones route 53 gave us.

-- 

hybrid dns

route 53 resolver answers dns queris for
* local domain names for ec2 instances
* records in private hosted zones
* records in public name servers

hybrid dns: resolving dns queries beween vps (route 53 resolver) and networks (other dns resolvers)

networks can be: vpc itself / peered vps or on-premises network connected through direct connect or aws vpn

inbound endpoint = we create such an endpoint to be able to listen for dns queries from outside (on premises data center). basically resolve domain names for aws resources (ec2 instances) and records in private hosted zones

outbound endpoint => the oppsosite. the resolver endpoint will forward the dns query to the dns resolvers on the onpremises data center.
