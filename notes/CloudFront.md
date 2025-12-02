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

Basically users try to fetch content in each edge location, cloudfront see if it has it cached, if not it fetches it from S3, and cache it in the edge location. if a user in the same edge location ask for the same content it will be faster to load because cloudfront has it in the cache of the same edge location

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