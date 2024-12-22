# SQS
## Overview
- Many producer can send messages to the SQS Queue and multiple consumers could poll messages.
- Fully managed service, used to decouple applications

## Attributes
- Unlimited throughput, unlimited number of messages in queue
- Default retention of messages: 4 days, maximum of 14 days
- Low latency (<10ms on publish and receive)
- Limitation of 256KB per message sent
- Can have duplicated messages (at least once delivery, occasionally)
- Can have out of order messages (best effort ordering)

## How it works?
- Produced to SQS using the SDK
- The message is persisted in SQS until a consumer deleted it
- SQS standard: unlimited throughput

## Auto Scaling Group for EC2 Instances
- Listen for specific events from CloudWatch metric to automatically scale the group of EC2

## Security
- Encryptions
    - In-flight encryption using HTTPS API
    - At-rest encryption using KMS keys
    - Client-side encryption if the client wants to perform encryption/decryption itself
- Access Controls: IAM policies to regulate access to the SQS API
- SQS Access Policies (similar to S3 bucket policies)
    - Useful for cross-account access to SQS queues
    - Useful for allowing other services (SNS, S3,...) to write to an SQS Queue

## Message Visibility Timeout
- After a message is polled by a consumer, it becomes invisible to other consumers
- By default, the message Visibility timeout is 30 seconds
- That means the message has 30 seconds to be processed
- After the message visibility timeout is over, the message is visible in SQS
- If a message isn't processed within the visibility timeout, it will be processed twice
- A consumer could call the ChangeMessageVisibility API to get more time
- If visibility timeout is high (hours), and consumer creashed, re-processing will take time
- If visibility timeout is too low (seconds), we may get duplicates

## Long Polling
- When a consumer requests messages from the queue, it can optionally wait for messages to arrive if three are none in the queue
- This is called Long polling
- LongPolling decreases the number of API calls made to SQS while increasing the efficiency and reducing latency of your application
- The wait time can be between 1 seconds to 20 seconds (20 seconds preferrable)
- Long Polling is preferrable to short Polling
- Long Polling can be enabled at the queue level or at the API level using WaitTimeSeconds

## Types of Queues
| Feature          | Standard Queue                                                     | FIFO Queue                                                      |
|------------------|---------------------------------------------------------------------|-----------------------------------------------------------------|
|**Availability**|Available in all regions|Available in a few regions (US East, EU, Asia Pacific, etc.)|
|**Throughput**|Unlimited – Can handle a large number of transactions per second|High – Supports up to 3,000 messages per second with batching|
|**Delivery**|At least once – Messages might be delivered more than once|Exactly once – Each message is delivered only once|
|**Ordering**|Best effort – Messages may arrive out of order|Strict – Messages arrive in the exact order they are sent|
|**Use Case**|Ideal for high throughput applications|Ideal for applications where message order matters|

## Tips for Auto Scaling
- When the number of EC2 instances (or other processor instances) increases, the transactions to database will be also increased.
SQS as a buffer to database writes is a good solution for this problem. Because SQS Queue is infinitely scalable.
This solution is only work if the client isn't impact by latency of writing to DB.

# SNS (Simple Notification Service)
## Overview
- A web service follow pub-sub messaging paradigm that makes it easy to set up, operate, and send notifications from the cloud.
- SNS is used for sending a message to many receivers.
- The event producer only sends message to one SNS topic.
- As many event receivers as we want to receive the notifications from SNS topic.
- Each subscriber to the topic will get all the messages (not: new feature to filter messages).
- Up to 12,500,000 subscriptions per topic.
- 100,000 topics limit.
- Many AWS services can send data directly to SNS for notifications

## Types
- FIFO: Destination is only SQS
- Standard: Destination could be email, SMS, Lambda, SQS, mobile application endpoints, HTTP, HTTPS.

## How to publish
- Topic publish (using the SDK)
    - Create a topic
    - Create a subscription (or many)
    - Publish to the topic
- Direct Publish (for mobile apps SDK)
    - Create a platform application
    - Create a platform endpoint 
    - Publish to the platform endpoint
    - Works with GG GCM, Apple APNS, Amazon ADM

## Security
### Encryption
- In-flight encryption using HTTPS API
- At-rest encryption using KMS keys
- Client-side encryption if the client wants to perform encryption/decryption itself

### Access Controls
IAM policies to regulate access to the SNS API
### SNS Access Policies (similar to S3 bucket policies)

## Fan Out pattern with SQS
- Push once in SNS, receive in all SQS queues that are subscriber
- Fully decoupled, no data loss
- SQS allows for : data persistence, delayed processing and retries of work
- Ability to add more SQS subscribers over time
- Cross-Region Delivery: works with SQS Queues in other regions.

### Example Features
- Combine with S3 Event Notifications to listen to S3's events changes and fan-out all those events to multiple SQS
- SNS to Amazon S3 through Kinesis Data Firehose (Preprocess data) and then save the result to supported KDF's destinations
- Support FIFO as FIFO SQS as well: Ordring by Message group ID, deduplication using a deduplication ID
    - Can have SQS standard and FIFO queues as subscribers
- Message Filtering: Based on the message to deliver to the correct destination. (I.e deliver to subscription queue if the message contain subscription fields.)


# Amazon MQ
## Overview
- Amazon MQ is a managed message broker service that makes it easy to migrate to AWS and operate message brokers in the cloud
- Supports industry-standard APIs and protocols like JMS, NMS, AMQP, STOMP, MQTT, and WebSocket
- Managed Apache ActiveMQ or RabbitMQ engine
- Doesn't scale as much as SQS/SNS but supports traditional protocols
- Runs on servers, can run in Multi-AZ with failover
- Has both queue feature (~SQS) and topic features (~SNS)

## Use Cases
- When you need to migrate applications using traditional message brokers to AWS without rewriting the messaging code
- Perfect for enterprise applications that need to migrate to the cloud
- Supports queue and pub/sub patterns

