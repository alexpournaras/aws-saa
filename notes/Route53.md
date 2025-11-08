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