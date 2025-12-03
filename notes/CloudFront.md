# AWS CloudFront

* Content Delivery Network (CDN)
* improves read performance, content is cached at edge locations
* improves user experience because of faster loading
* ddos protection because its worldwide, integration with Shield, AWS web application firewall

cloudfront origins:
* s3 bucket
    - distributing files and caching them at the edges
    - for uploading to s3 through cloudfront
    - secured using Origin Access Controll (OAC)

* vpc origin
    - applications hosted in vpc private subnets
    - application load balancer/network load balancer/ ec2 instances

* custom origin (http)
    - s3 website
    - any public http backend we want

Basically users try to fetch content in each edge location, cloudfront see if it has it cached, if not it fetches it from S3, and cache it in the edge location. if a user in the same edge location ask for the same content it will be faster to load because cloudfront has it in the cache of the same edge location.

* to enforce users to access the website only through cloudfront (private s3 website), we configure cloudfront distribution and create an OAC, then update s3 bucket policy to only accept requests from this distribution

cloudfront vs s3 cross region replication
* cloudfront
    - global edge network (216 points of presence)
    - files are cached with a TTL
    - great for static content that must be available everywhere

* s3 cross region replication
    - must be setup for each region we want the replication
    - files are updated in near real time
    - read only
    - great for dynamic content that needs to be available at low-latency in few regions

--

Cloudfront - ALB or EC2 as origin using VPC Origins
* allows you to deliver content from your applications hosted in your VPC private subnets (no need to expose them on the internet)
* Deliver traffic to private ALB, NLB, EC2 instances.

users -> cloudfront -> vpc origin -> ec2 instance in private vpc

* the old way was with security groups that only allowed cloudfront ips

--

cloudfront geo restriction
* we can restrict who can access our distribution
    - Allowlist: list of countries of approved countries
    - Blocklist: list of counties where access is blocked/banned

* the 'country' is determined by IP
* use case: copyright laws to controll access to content

--

cloudfront pricing

* the cost of data out (send) per edge location varies. (usa and europe is cheaper than india or china)
* we can reduce the number of edge locations for some cost reduction
    - price class All (all locations, best performance)
    - price class 200 (most regions, excludes the most expensive)
    - price class 100 (only the cheaper, usa + europe)

--

cloudfront cache invalidation
* cloudfront doesnt know if we change something in our content, it will refetch the content after the TTL has expired
* but we can enforce (bypass ttl) to do entire or partial cache refrech with cloudfront invalidation
* we can invalidate all files (*) or specific path (/images/*)

--

global accelerator

* unicast IP: one server holds one IP address
* anycast IP: all servers hold the same IP adress and the client is routed to the nearest one

* global accelerator use anycast ip by prividing 2 anycast ips for our application
* the anycast ip send traffic directly to edge locations
* the edge locations send the traffic to our application in private/internal network (faster)
* works with elastic IP, ec2, alb, nlb, public or private
* routing to lowest latency and fast regional failover
* the ip doesnt change so no problems with client cache
* health checks, great for disaster recovery and failover
* only 2 external IPs need to be whitelisted
* ddos protection thanks to AWS Shield
* client affinity: our the same client we want to go to the same backend instance

so in comparison:
Global Accelerator vs CloudFront

cloudfront:
* improves performance for cacheable content (images, videos)
* dynamic content such as API acceleration and dynamic site delivey
* IMPORTANT: content is served at the edge location!!!

global accelerator:
* improves performance for a wide range of applications over tcp or udp
* IMPORTANT: proxing packets at the edge to applications running in one or more aws regions
* good fit for non-http use cases (gaming, iot, voice over ip)
* good for http use cases that require static ip address (important!!!)
* good for http use cases that require deterministing and fast regional failover

so basicaly, cloudfront has cached our application in the edge locations (no need to send traffic to application if its cached), while global accelerator use the edge locations to send the traffic internal to our application in aws.
