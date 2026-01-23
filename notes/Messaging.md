# Communication

2 patterns of communication

synchronous = app connect to another app and talk
asynchronous = apps connect to a queue. one app adds to queue, the other gets from the queue.
basically with the queue, we decouple applications and then we can scale accordingly

amazon sqs (Simple Queue Service)

Many "producers" that send messages to the queue
Many "consumers" that poll queue for messages

* unlimited throughput, unlimited number of messages in queue
* default retention of messages: 4 days, maximum 14 days
* low latency (<10 ms on publis and receive)
* limitation on 1024 KB per message max

producers connect with SDK (SendMessage API)
the message is presisted in SQS until consumer deletes it (or max number of days for retention)

consumers: can be running on ec2 instances, servers, aws lambda etc.
* poll sql messages (can revceive up to 10 messages at a time)
* then process the mssages
* then use SDK (DeleteMessage API)

we can have multiple consumers and multiple produces, therefore: 

* could be duplicate messages (at least once delivery)
* could have out of order messages (best-effort message ordering)
* consumers receive and process messages in parallel
* we can scale consumers horizontally to imporve throughtput of processing

example scenario:
we can use cloudwatch metrics and cloudwatch alarms to work with an autoscaling group that whenever we have a spike on messages in our queue, the auto scaling group can create more backends (consumers) to poll the messages.

---

sqs message visibility timeout

* after a message is polled by a consumer it becomes invisible to other consumers
* by default for 30 seconds
* after 30 seconds, message becomes again visible in sqs
* to extend the timout while processing, we have to make a call to the ChangeMessageVisibilty API to get more time

amazon sqs - long polling

* when consumers polls messages and there arent any in the queue, it can optionally wait more time.
* long polling decreases the number of api calls made to sqs, increasing efficinecy and reducing latency of our application
* the wait time can be between 1 sec - 20 sec (20 sec preferable)
* long polling is preferable to short polling
* long polling can be enabled at the euque level or at the api level using WaitTimeSeconds API

amazon sqs - fifo queue

fifo = first in first out (ordering of messages in queue)

* limited thorughput 300 msg/s (3000 msg/s with batching)
* exactly-once send capability (by removing duplicates using deduplication ID = basically it auto removes duplicated ids within 5 minutes) (content-based deduplication)
* ordering by message group ID (all messages in the same group are ordered)

---

amazon SNS (Simple Notification Service)

to send one message to many receivers there is the pub/sub pattern.

* the "event producer" ony sends a message/notification to one SNS topic and all subscribers "event receivers" for this topic will receive it. (we can filter here)
* up to 12 millions subscribtions per topic
* 100.000 topics limit per account

subscribers can be: emails, SMS, mobile notifications, http(s) endpoints, SQS, Lambda, Kinesis Data Firehose, etc.

the producers can be any service that can produce notifications. like cloudwatch alarms, auto scaling group, s3 bucket events, rds events, aws budgets, lambdas, etc.

to publis we need SDK
* create topic, create subscriptions, publish topic
* "direct publish" => its for mobile apps SDK
    - create platform application, create platform endpoint, publish to platform endpoint
    - works with google gcm, apple apns, amazon adm (different ways of our mobile applications to receive notifications)

message filtering
* json policy used to filter messages sent to sns topic's subscriptions

---

sns + sqs fan out pattern

* push once in sns topic, many sqs queues are subscribed to that topic 
* data persistences, delayed processing, retries of work
* ability to add more sqs subsribers over time
* make sure we configured the access policies
* cross-region delivery works with sqs queues in other regions
* other services can also subscribe to this topic and apply the fan out pattern. like lambda functions or kinesis data firehose (kdf)

---

amazon sns fifo topic

fifo = first in first out (ordering of messages in topic)

* limited thorughput 300 msg/s (3000 msg/s with batching)
* exactly-once send capability (by removing duplicates using deduplication ID = basically it auto removes duplicated ids within 5 minutes) (content-based deduplication)
* ordering by message group ID (all messages in the same group are ordered)
* only sqs as subscribers but can use both standard and fifo queues.

--- 

SQS & SNS security

* HTTPS API
* KMS keys if needed
* client-side encryption if clients wants to perform encryption decryption themselfs

access controls: iam polices for SQS/SNS api
SQS/SNS Access Policies (similar to S3 bucket policies)
* usefull for cross-acount access
* usefull for allowing other services (SNS/SQS, S3 etc) to write to an SQS Queue / SNS Topic

---

amazon kinesis data streams

* collect and store streaming data in *real-time*
real time data (iot devices) -> producers (applications, kinesis agent) -> kinesis data streams -> consumers (applications, lambda, amazon data firehose, etc.)
* retention between up to 365 days
* ability to reprocess (replay) data by consumers
* data can't be deleted from kinesis until expiration
* data up to 10 mib (we want lots of small real time data)
* kms and https encryption
* data ordering guarantee for data with the same "partition id"
* libraries: kinesis producer library (kpl), kinesis client library (kcl)

* on-demand mode:
    - no need to provision or manage capacity of shards
    - default capacity provisioned (4mb/s in or 4000 records per second)
    - scales automatically based on observed throughput peak during last 30 days
    - pay per stream per hour & data in/out per GB

* provisioned mode:
    - choose number of shards
    - each gets 1mb/s in (or 1000 records per second)
    - each gets 2mb/s out
    - scale manually
    - pay per shard provisioned per hour

---

