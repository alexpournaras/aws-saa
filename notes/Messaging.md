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

sqs security

* HTTPS API
* KMS keys if needed
* client-side encryption if clients wants to perform encryption decryption themselfs

access controls: iam polices for sqs api
SQS Access Policies (similar to S3 bucket policies)
* usefull for cross-acount access
* usefull for allowing other services (SNS, S3 etc) to write to an sqs queue

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