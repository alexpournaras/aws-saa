Amazon EC2

Elastic Compute Cloud
Infrastructure as a Service

* Renting virtual machines (EC2)
* Storing data on virtual drives (EBS)
* Distributing load across machines (ELB)
* Scaling the services using an auto-scaling group (ASG)

Configuration
* OS: linux, windows, macos
* cpu
* ram
* storage:
    - network-attached: EBS & EFS
    - hardware (EC2 instance store)
* Network: speed, public IP address
* Firewall rules: security group
* Bootstrap script configure at first launch: EC2 User Data

With EC2 User Data:
bootrap our instances using EC2 User Data script
* execute commands when machine starts/automate boot. example
    - installing updates
    - installing software
    - download etc etc

AMI:OS  image

key pair : RSA type and .pem files

when instance is stopped, you dont pay for it

when restarting the instance the public ip might change!!!

instance types:
general purpose => balanced
    - compute, memory, networking
    example (t2 micro)

compute optimized => high cpu => instance classes starts with C
    - batch processing workloads
    - media transcoding
    - high performance web servers
    - high performance computing (hpc)
    -sientific modeling & machine learning
    - dedicated gaming servers

memory optimized => process large data sets in memory
    use the R in front of instances names
    - high performance relational-non/relational databases
    - distrubuted web scale cache stores
    - in-memory databases optimized for Business instelligence
    - applications performing real time processing of big unstructured data

storage optimized => starts with I or D or H1
    => high read and write to large data sets on storage
    - high frequency online transaction processign systems OLTP
    - relationanl & nosql databases
    - cache for inmemory databases for example redis
    - data warehousing applications
    - distrubuted file systems

accelerated computing
hpt otpimized
instance features
measuring instance performance

m5.2xlarge
m = instance class
5 = generation/version
2xlarge = size of instance for that class

security groups = firewall
only contain allow rules
security groups rules can refernce by IP or by other security group
locked down to a region/vpc combination
is OUTSIDE of the EC2

timeout = security group issue
connection refused = application error

all inbound traffic is blocked
all outbound traffic is authorised

ports:
22 = ssh (secure shell)
21 = FTP file transfer protocol
22 = STFP secure >> (its happeing with ssh thats why 22)
80 = HTTP
443 = HTTPS
3389 = RDP (remote destkop protocol for windows)

chmod 044 key.pem
ssh -i key.pem ec2-user@3.250.26.200

EC2 Instance Connect = browser based SSH inside the ec2 instance (needs the ssh rule of course)

never configure aws inside the ec2 instance
instance -> actions -> security -> modify IAM role
then if we give permissions, we can do:
aws iam list-users







