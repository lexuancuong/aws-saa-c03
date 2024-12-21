# AWS SAA C03

## Organization Management

### IAM

#### Overview
- Identity and Access Management Global Service  
- Root account is created by default, shouldn't be used or shared  
- Group only contains users, not other groups  
- Permissions can be assigned directly but should be attached to users via groups  
- IAM policy could inherit permissions if the user belongs to multiple groups  

#### Configuration
- **SID**: Statement ID  
- Can configure policy with JSON  
- **Effect**: Whether the statement allows or denies  
- **Principal**: Account/user/role to which this policy applies  
- **Action**: List of actions this policy allows or denies  
- **Resource**: List of resources to which the actions apply  
- **Condition**: Specifies when this policy is in effect  

#### MFA
- Virtual app  
- Security key  
- TOTP  

#### Tips
- On the AWS console UI, the root account doesn't have an IAM user icon on the top right.

## Storage

### S3
#### Overview
- Amazon S3 is one of the main building blocks of AWS

#### Use cases
- Backup and Storage
- Disaster Recovery
- Archive
- Hybrid Cloud Storage
- Application hosting
- Media hosting
- Data lakes and big data analytics
- Software delivery
- Static website

#### Components
##### Buckets
- As root directories
- Must have a globally unique name (across all regions all accounts)
- Buckets are defined at the region level

##### Objects
- As a file
- Have a key as a FULL path
- There's no concept of directories within buckets (although the UI will trick you to think otherwise)
- Max object size is 5TB
- If uploading more than 5GB, must use "multi-part upload"
- Properties
  - Metadata
    - List of text key/value pairs - system or user metadata
  - Tags
    - Unicode key/value pair - up to 10 (useful for security/lifecycle)
  - Version ID (If versioning is enabled)

#### Security
##### User-Based
- IAM Policies - which API calls should be allowed for a specific user from IAM

##### Resource-Based
- Bucket Policies
  - Definition
    - bucket wide rules from the S3 console - allows cross account
  - JSON based policies
    - Resources
      - buckets and objects
    - Effect: Allow / Deny
    - Actions: Set of API to allow or deny
    - Principal: The account or user to apply the policy to
  - Usage
    - Grant public access to the bucket
    - Force objects to be encrypted at upload
    - Grant access to another account (Cross Account)
- Object Access Control List (ACL)
  - finer grain (can be disabled)
- Bucket Access Control List

##### Note
- an IAM principal can access an S3 object if the user IAM permissions allow it or the resource policy allows it
- Could encrypt objects in Amazon S3 using encryption keys
- Configure Bucket Policy Handons
  - use awspolicygen to generate the JSON template to configure S3
  - If you get 403 forbidden error, ensure that the bucket policy allows public reads

#### Versioning
##### Scope
- It is enabled at the bucket level

##### Reasons
- Protect against unintended deletes
- Easy rollback to previous version

##### Notes
- Any file that is not versioned prior to enabling versioning will have version "null"
- Suspending versioning does not delete the previous versions

#### Replication
- Must enable Versioning in source and destination buckets

##### Types
- Cross Region Replication (CRR)
  - Compliance, lower latency access replication across accounts
- Same Region Replication (SRR)
  - Log aggregation, live replication between production and test accounts

- Buckets can be in different AWS accounts
- Copying is asynchronous
- Must give proper IAM permissions to S3

##### Notes
- After enabling Replication, only new objects are replicated
  - You can replicate existing objects using S3 Batch Replication (for failed replications as well)
- Can replicate delete markers from source to target
- Deletions with a version ID are not replicated (to avoid malicious deletes)
- There is no chaining of replication

#### Storage Classes
##### Standard - General Purpose
- 99.99% Availability
- Used for frequently accessed data
- Low latency and high throughput
- Sustain 2 concurrent facility failures
- Use cases: Big data analytics, mobile & gaming applications, content distribution...

##### Infrequent Access
- Used for less frequently accessed but required rapid access when needed
- Lower cost than S3 standard
- S3 Standard - Infrequent Access
  - 99.9% availability
  - Use cases: Disaster recovery, backups
- S3 One Zone - Infrequent Access
  - High durability 99.9999999% in a single AZ, data lost when AZ is destroyed
  - 99.5% availability
  - Use cases: Storing secondary backup copies of on-premise data or data you can recreate

##### Glacier
- Low-cost object storage meant for archiving/backup
- Pricing: price for storage + object retrieval cost
- S3 Glacier Instant Retrieval
  - Millisecond retrieval great for data accessed once a quarter
  - Minimum storage duration of 90 days
- S3 Glacier Flexible Retrieval
  - Expedited (1 to 5 minutes), Standard (3 to 5 hours), Bulk (5 to 12 hours)
- S3 Glacier Deep Archive
  - Standard (12 hours), bulk (48 hours)
  - Minimum storage duration of 180 days

##### S3 Intelligent Tiering
- Small monthly monitoring and auto-tiering fee
- Move objects automatically between Access Tier based on usage
- There are no retrieval charges in S3 intelligent tiering

##### Lifecycle Rules
- Transition Actions
  - Description
    - Configuration objects to transition to another storage class
  - Examples
    - Move objects to standard IA class 60 days after creation
    - Move to Glacier for archiving after 6 months
- Expiration actions
  - Description
    - Configure objects to expire (delete) after some time
  - Examples
    - Access log files can be set to delete after 365 days
    - Can be used to delete old versions of files (if versioning is enabled)
    - Can be used to delete incomplete Multi-Part uploads
- Configurable objects
  - Rules can be created for a certain prefix (example: s3://mybucket/mp3)
  - Rules can be created for certain objects tags (example: Department Finance)

##### Notes:
- Can move between classes manually or using S3 Lifecycle configurations

#### S3 Requester Pays
- In general, bucket owners pay for all Amazon S3 storage and data transfer cost associated with their bucket
- With requester pays buckets, the requester instead of the bucket owner pays the cost of the request and the data download from the bucket
- Helpful when you want to share large datasets with other accounts
- The requester must be authenticated to AWS

#### S3 Event Notifications
##### Type
- S3:ObjectsCreated
- S3:ObjectRemoved
- Object name filtering possible (*.jpg)
- Use case: generate thumbnails of images uploaded to S3
- Can create as many S3 events as desired
- Asynchronous events
- Amazon EventBridge support delivering all messages to over 18 AWS services as destinations
- Can also configure to send S3 events to SNS, SQS and lambda
  - Need to set up destination's policy to allow S3 notification send message to

#### Baseline Performance
- Amazon S3 automatically scales to high request rates (latency 100-200ms)
- Your application can achieve at least 3500 PUT/COPY/POST/DELETE or 5,500 GET/HEAD requests per second per prefix in a bucket
- There are no limits to the number of prefixes in a bucket
- Example (object path => prefix)
  - bucket/prefix/foldername/filename
- If you spread reads across all four prefixes evenly, you can achieve 22,000 requests per second for GET and HEAD

##### Multi-Part upload
- recommended for files > 100MB, must use for files > 5GB
- Can help parallelize uploads (speed up transfers)

##### Transfer Acceleration
- Increase transfer speed by transferring file to an AWS edge location which can forward quickly the data to the S3 bucket in the target region
- Compatible with multi-part upload

##### Byte-Range Fetches
- Parallelize GETs by requesting specific byte ranges
- Better resilience in case of failures
- => Can be used to speed up downloads

#### Batch Operations
- Perform bulk operations on existing S3 objects with a single request
  - Modify object metadata and properties
  - Copy objects between S3 buckets
  - Encrypt un-encrypted objects
  - Modify ACLs tags
  - Restore objects from S3 Glacier
  - Invoke Lambda function to perform custom action on each object
- A job consists of a list of objects, the action to perform and optional parameters
- Batch Operation manages retries, tracks progress, sends completion notifications, generates reports
- You can use S3 Inventory to get object list and use S3 Select to filter your objects

#### Storage Lens
- Understand, analyze and optimize storage across entire AWS Organization
- Discover anomalies, identify cost efficiencies, and apply data protection best practices across entire AWS Organization (30 days usage and activity metrics)
- Aggregate data for organization, specific accounts regions buckets or prefixes
- Default dashboard or create your own dashboards
  - Visualize summarized insights and trends for both free and advanced metrics
  - Default dashboard shows multi-region and multi-account data
  - Preconfigured by Amazon S3
  - Can't be deleted but can be disabled
  - Metrics
    - Summary metrics
      - General insights about your S3 storage
      - StorageBytes, ObjectCount
      - Use cases: Identify the fastest-growing (or not used) buckets and prefixes
    - Data-protection metrics
    - Cost-optimization metrics
    - Access-management Metrics
      - S3 object ownership
    - Event Metrics
    - Performance Metrics
    - Activity Metrics
    - Detailed status code metrics
    - Free metrics
    - Advanced Metrics and Recommendations
- Can be configured to export metrics daily to an S3 bucket

#### Object Encryption
##### Server-Side Encryption (SSE)
- With Amazon S3-managed keys (SSE - S3) Enabled by Default
  - Encryption using keys handled, managed and owned by AWS
  - Object is encrypted server-side
  - Encryption type is AES-256
  - Must set header "x-amz-server-side-encryption": "AES256"
  - Enabled by default for new buckets and new objects
- With KMS Keys stored in AWS KMS (SSE-KMS)
  - Handled and managed by AWS KMS (Key management service)
  - KMS advantages: user control + audit key usage using CloudTrail
  - Object is encrypted server side
  - Must set header "x-amz-server-side-encryption": "aws:kms"
  - Limitations
    - If you use SSE-KMS, you may be impacted by the KMS limits
    - When you upload, it calls the GenerateDataKey KMS API
    - When you download, it calls the Decrypt KMS API
    - Count towards the KMS quote per second
    - You can request a quote increase using the Service Quotas Console
- with Customer-provided keys (SSE-C)
  - The keys managed by the customer outside of AWS
  - S3 does not store the encryption key you provide
  - HTTPS must be used
  - Encryption key must be provided in HTTP headers, for every HTTP request made
  - To read that files the user needs to provide the keys as well

##### Client-side encryption
- Use client libraries such as Amazon S3 Client-Side encryption library
- Clients must encrypt data themselves before sending to Amazon S3
- Clients must decrypt data themselves when retrieving from Amazon S3
- Customer fully manages the keys and encryption cycle

##### Notes
- Bucket policies are evaluated before default encryption
- Could force encryption using a bucket policy and refuse any API call to PUT an S3 object without encryption headers (SSE-KMS or SSE-C)

#### S3 CORS
- If a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers
- CORS issues usually happen because the objects could be stored in different S3 buckets (=> different URLs)
- Need to add Access-Control-Allow-Origin to allow this cross buckets access
  - Configuring involves AllowedHeaders, AllowedMethods and AllowedOrigins, ExposeHeaders, MaxAgeSeconds

#### Amazon S3 - MFA Delete
##### Definition
- force users to generate a code on a device before doing important operations on S3

- MFA will be required to permanently delete an object version
- Suspend versioning on the bucket
- MFA won't be required to
  - Enable Versioning
  - List deleted versions
- Only the bucket owner(root account) can enable/disable MFA Delete

#### Access Logs
- For audit purpose, you may want to log all access to S3 buckets
- Any request made to S3, from any account authorized or denied will be logged into another S3 bucket
- That data can be analyzed using data analysis tools...
- The target logging bucket must be in the same AWS region

##### Warnings
- Do not set your logging bucket to be the monitored bucket
- It will create a logging loop and your bucket will grow exponentially

#### Pre-signed URLs
- Generate pre-signed URLs using S3 console, AWS CLI or SDK
- Users given a pre-signed URL inherit the permissions of the user that generated the URL for GET/PUT
- Use cases
  - Allow only logged-in users to download a premium video from your S3 bucket
  - Allow an ever-changing list of users to download files by generating URLs dynamically
  - Allow temporarily a user to upload a file to a precise location in your S3 bucket

#### Glacier Vault Lock
- Adopt a WORM (Write Once Read Many) model
- Create a Vault Lock Policy
- Lock the policy for future edits (can no longer be changed or deleted)
- Helpful for compliance and data retention

#### S3 Object Lock
- Versioning must be enabled
- Adopt a WORM
- Block an object version deletion for a specified amount of time
- Retention Modes
  - Compliance
    - Object versions can't be overwritten or deleted by any user, including the root user
    - Objects retention modes can't be changed and retention periods can't be shortened
  - Governance
    - Most users can't overwrite or delete an object version or alter its lock settings
    - Some users have special permissions to change the retention or delete the object
- Retention Period
  - Protect the object for a fixed period, it can be extended
- Legal Hold
  - Protect the object indefinitely, independent from retention period
  - Can be freely placed and removed using the s3:PutObjectLegalHold IAM permission

#### Access Points
- Manage the permissions of group of people or departments
- Configure via Policy setup
- Each access point has its own DNS name
- an access point policy (similar to bucket policy) - manage security at scale
- We can define the access point to be accessible only from within the VPC
- You must create a VPC Endpoint to access the Access Point
- The VPC Endpoint Policy must allow access to the target bucket and Access Point

#### S3 Object Lambda
- Use AWS Lambda Functions to change the object before it is retrieved by the caller application
- Only one S3 bucket is needed, on top of which we create S3 Access Point and S3 Object Lambda Access Points
- Use cases
  - Converting data format
  - Resizing and watermarking images on the fly using caller-specific details, such as the user who requested the object

### EBS
#### Overview
- Network USB Stick
- EBS could attach to 1 EC2 instance at 1 time
- Could configure to terminate together with EC2 or not
- One EC2 instance could attach multiple EBS
- Must be in the same AZ with EC2

#### EBS Snapshots
- Make a backup of EBS volume at a point in time
- Not necessary to detach volume to do snapshot, but recommend
- Can copy snapshot across AZ or Region
- EBS Snapshot Archive Tier is 75% cheaper
- Take within 24 to 72 hours for restoring the archive
- Recycle bin
  - Can setup retention 1 day -> 1 year
  - Fast snapshot restore
  - Force to install all from snapshot

#### EBS Volume Types
- gp2/gp3 (SSD)
  - General purpose SSD volume that balances price and performance for a wide variety of workloads (Can be root volume)
- io1/io2 (SSD)
  - Highest performance SSD for mission critical low latency or high through workloads (can be root volume)
- st1 (HDD)
  - low cost HDD volume design for frequently accessed throughput intensive workloads
- sc1 (HDD)
  - lowest cost HDD volume designed for less frequently accessed workloads

### EFS
#### Overview
- Network file system for Linux instances, POSIX filesystem

### EC2 Instance Store
- EBS volumes are network drives with good but limited performance
- If you need a high performance hardware disk, use EC2 instance store

#### Use cases
- Significantly better for I/O performance
- EC2 Instance Store lose their storage if they are stopped
- Good for buffer scale/scratch data/ temporary content
- Risk of data loss if hardware fails
- Backup and Replication are your responsibility

### FSx
#### Overview
- Launch 3rd party high-performance file systems on AWS
- Fully managed service

#### Types
##### Windows File Server
- Can be mounted to Linux EC2 instance
- Supports SMB protocol and Windows NTFS
- Microsoft Active Directory integrations, ACLs, user quotas
- Supports Microsoft's Distributed File System (DFS) Namespaces (group files across multiple FS)
- Scale up to 10s of GB/s, millions of IOPS, 100s PB of data
- Storage Options
  - SSD - latency sensitive workloads (databases, media processing, data analytics,...)
  - HDD - broad spectrum of workloads
- Can be accessed from your on-premises infra (VPN or Direct Connect)
- Can be configured to be Multi-AZ (high availability)
- Data is backed-up daily to S3

##### Lustre
- Overview
  - Is a type of parallel distributed file system, for large-scale computing
  - The name Lustre is derived from "Linux" and "cluster"
- Performance
  - Machine Learning, High Performance Computing (HPC)
  - Video Processing, Financial Modeling, Electronic Design Automation
  - Scales up to 100s GB/s, millions of IOPS, sub-ms latencies
- Storage Options
  - SSD - low latency, IOPS intensive workloads, small and random file operations
  - HDD - throughput-intensive workloads, large and sequential file operations
- Integrate with S3
  - Can read S3 as a file system through FSx
  - Can write the output of the computations back to S3 through FSx
- Can be used on premises server via VPN or Direct Access
- Deployment Options
  - Scratch File System
    - Temporary storage
    - Data is not replicated (doesn't persist if file server fails)
    - High robust 6x faster
    - Usage: short-term processing optimize cost
  - Persistent File System
    - Long-term storage
    - Data is replicated within same AZ
    - Replace failed files within minutes
    - Usage: long term processing, sensitive data

##### NetApp ONTAP
- Overview
  - Managed NetApp ONTAP on AWS
  - File System compatible with NFS, SMB, iSCSI protocol
  - Move workloads running on ONTAP or NAS to AWS
  - Works with
    - Linux
    - Windows
    - MacOS
    - VMware Cloud on AWS
    - Amazon Workspaces and AppStream 2.0
    - Amazon EC2, ECS and EKS
  - Storage shrinks or grows automatically
  - Snapshots replication low-cost compression and data de-duplication
  - Point-in-time instantaneous cloning (helpful for testing new workloads)

##### OpenZFS
- managed OpenZFS file system on AWS
- File System compatible with NFS
- Move workloads running on ZFS to AWS
- Works with:
  - Linux
  - Windows
  - MacOS
  - VMware Cloud on AWS
  - Amazon workspaces and app stream 2.0
  - Amazon EC2, ECS and EKS
  - Up to 1mil IOPS with < 0.5ms latency
  - Point-in-time instantaneous cloning (helpful for testing new workloads)

### Snowball
#### Overview
- A offline service that allows users to transfer large amounts of data to their AWS cloud, even in areas with limited or no network connectivity. It's useful for businesses that collect data in remote locations, such as those in the architecture construction, energy and gaming industries

#### Types
- Snowcone
  - 8 TB HDD -> 14 TB SSD
- Snowball Edge
  - 80TB-210TB

#### Edge computing
- Process data while it's being created on an edge location

#### How it works
1. Order the device via AWS management console
2. Receive and copy data
3. Ship back
4. Data Import by AWS

#### Variants
- Snowball Edge Storage Optimized
  - Primarily for bulk data transfer and storage
- Snowball Edge Compute Optimized
  - Include powerful compute capability for edge processing
- Snowcone
  - A smaller, more portable device for small-scale data transfers edge computing

### AWS Storage Gateway
#### Overview
- Hybrid cloud storage that bridges your on-premises infrastructure with AWS storage services

#### Use cases
- Enables you to use AWS cloud storage for backups, archiving, and data tiering while keeping your on-premises applications running seamlessly

#### Types
- File Gateway
  - Provides an interface for applications to access files via standard protocols like NFS or SMB
  - Data is stored in Amazon S3 as objects
- Tape Gateway
  - Virtual tape library for backup applications
  - Data is stored in S3 and archived to S3 Glacier or Glacier Deep Archive
- Volume Gateway
  - Cached mode: Frequently accessed data is cached locally with all data stored in S3
  - Store mode: Entire dataset is stored on-premises, with backups sent to EBS

### AWS Data Sync
- Move large amount of data to and from on-premises/other cloud to AWS (need agents). Or AWS to AWS (different storage services) - no agent needed.
- Can synchronize to S3 (any storage class - including Glacier), Amazon EFS
- Replication task can be scheduled hourly, daily or weekly
- File permissions and metadata are preserved (NFS POSIX, SMB,...)

## Networking

### ELB


### Route 53
#### Overview
- a domain registrar
  - Allow you to buy new domain like other provider i.e GoDaddy
- the only service has 100% SLA
- Ability check health for resources
- is a global service

#### Configuration Fields
##### Domain/Sub Domain Name

##### Record Type
- A
  - maps a hostname to IPv4
  - Alias to an Internal AWS Services
    - Work for both root domains and non root domains
    - Maps a hostname to an AWS resource
    - Automatically recognizes changes in the resource's IP address
    - Can't set the TTL
    - Targets
      - Elastic Load Balancers
      - CloudFront Distributions
      - API Gateway
      - S3 websites
      - VPC Interface Endpoints
      - CANNOT set ALIAS RECORD for EC2 DNS name
- AAAA
  - maps a hostname to IPv6
- CNAME
  - maps a hostname to another hostname
  - Can't create a CNAME record for top node of a DNS namespace
    - It means that we cannot create CNAME record for root domain
- NS
  - Name servers for the hosted Zone
  - Control how traffic is routed for a domain
  - In case you buy domain from another 3rd party provider (i.e GoDaddy, could configure NS records on their sites to use Route 53 Name Servers)

##### Routing Policy
- Define how Route53 responds to DNS queries
- Supported Routing Policy
  - Simple
    - Route traffic to a single resource
    - Can specify multiple values in the same record
      - If multiple values are returned, a RANDOM one is chosen by the client
    - When alias enabled, specify only one AWS resource
    - Can't be associated with health check
  - Weighted
    - Control the percentage of the requests that go to each specific resource
    - Assign each record a relative weight. Weights don't need to sum up to 100
    - Use cases: Load balancing between regions, testing new application versions
    - Assign a weight of 0 to a record to stop sending traffic to a resource
    - If all records have weight of 0, then all records will be returned equally
  - Latency-based
    - Redirect to the resource that has the least latency close to us
    - **Latency is based on traffic between users and AWS Regions**
    - Germany users may be directed to the US (if that's the lowest latency)
    - Can be associated with health checks
  - Failover (Active-Passive)
    - There are many A/AAAA records with associated health checks From top to down, if the health check of the first A record fail, Route53 returns the next record in the list of failover records
  - Geolocation
    - This routing is based on user location (different from Latency-based)
    - Specify location by Continent, Country or by US State
    - Should create a default record in case there's no match on location
    - Can be associated with health checks
    - Useful in case prohibit users from specific country from accessing
  - Geoproximity
    - To shift more traffic to AWS regions or lat&long based on associated defined biases
  - IP-based Routing
    - Routing is based on clients' IP addresses
    - You provide a list of CIDRs for your clients and the corresponding endpoints/locations (user IP to endpoint mapping)
  - Multi-Value
    - Routing traffic to multiple resources
    - Route53 returns multiple values/resources
    - Can be associated with Health Checks (return only values for healthy resources)
    - Up to 8 healthy records are returned for each Multi-value query
    - Multi-Value is not a substitute for having an ELB

##### TTL
- The time for the client to cache the DNS result to avoid spamming to DNS resolver

#### Hosted Zones
- Public Hosted Zones
  - Contains records that specify how to route traffic on the Internet
- Private Hosted Zones
  - contain records that specify how you route traffic within one or more VPCs (private domain names)
  - Just allow private resources within a VPC access

#### Cost
- Pay 0.5$ per month per hosted zone
- Extra cost per number of requests

#### Tools
##### Commands
- nslookup: To find the associated IP with a domain
- dig: Show more information like TTL

#### Health Checks
- Only for public resources
- Types
  - Monitor an Endpoint
    - About 15 global health checkers will check the endpoint health
      - Interval 30 seconds (can set to 10 sec - higher cost)
      - Support HTTP, HTTPS and TCP
      - If >18% of health checkers report the endpoint is healthy, Route53 considers it Healthy.
      - Ability to choose which locations you want Route53 to use
    - Pass for 2xx and 3xx status codes
    - Health Checkers can be setup to pass/fail based on the first 5012 bytes of the response
    - Need to configure router/firewall to allow incoming requests from Route53 Health Checkers
  - Calculated health checks
    - Combine the results of multiple Health Checks into a single Health Check
    - Support logic like **or**, **and**, **not**.
    - Up to 256 child health checks
    - Specify how many of the health checks need to pass to make the parent pass
  - CloudWatch Metric and Alarm
    - Create health checks based on CloudWatch's alarms

### CloudFront
#### Overview
- Content Delivery Network
- Improve read performance, content is cached at the edge
- Improve user experiences
- 216 Point of Presence globally (edge locations)
- DDoS protections, integration with Shield, AWS Web Application Firewall.

#### Origins
- EC2 or ALB
- S3 bucket
  - For distributing files and caching them at the edge
  - Enhance security with CloudFront Origin Access Control (OAC)
  - CloudFront could be used as an ingress to upload file to S3
- Custom Origin (HTTP)
  - Application Load Balancer
  - EC2 Instance
  - S3 website
  - Any HTTP backend you want

#### Geo Restriction
- You can restrict who can access your distribution
  - Blocklist
    - Prevent your users from accessing your content if they're in one of the countries on a list of banned countries
  - AllowList
    - Allow your users to access your content only if they're

## Message Queue
### SQS
#### Overview
- Many producer can send messages to the SQS Queue and multiple consumers could poll messages.
- Fully managed service, used to decouple applications
#### Attributes
- Unlimited throughput, unlimited number of messages in queue
- Default retention of messages: 4 days, maximum of 14 days
- Low latency (<10ms on publish and receive)
- Limitation of 256KB per message sent
- Can have duplicated messages (at least once delivery, occasionally)
- Can have out of order messages (best effort ordering)
#### How it works?
- Produced to SQS using the SDK
- The message is persisted in SQS until a consumer deleted it
- SQS standard: unlimited throughput

#### Auto Scaling Group for EC2 Instances
- Listen for specific events from CloudWatch metric to automatically scale the group of EC2

#### Security
- Encryptions
    - In-flight encryption using HTTPS API
    - At-rest encryption using KMS keys
    - Client-side encryption if the client wants to perform encryption/decryption itself
- Access Controls: IAM policies to regulate access to the SQS API
- SQS Access Policies (similar to S3 bucket policies)
    - Useful for cross-account access to SQS queues
    - Useful for allowing other services (SNS, S3,...) to write to an SQS Queue
#### Message Visibility Timeout
- After a message is polled by a consumer, it becomes invisible to other consumers
- By default, the message Visibility timeout is 30 seconds
- That means the message has 30 seconds to be processed
- After the message visibility timeout is over, the message is visible in SQS
- If a message isn't processed within the visibility timeout, it will be processed twice
- A consumer could call the ChangeMessageVisibility API to get more time
- If visibility timeout is high (hours), and consumer creashed, re-processing will take time
- If visibility timeout is too low (seconds), we may get duplicates

#### Long Polling
- When a consumer requests messages from the queue, it can optionally wait for messages to arrive if three are none in the queue
- This is called Long polling
- LongPolling decreases the number of API calls made to SQS while increasing the efficiency and reducing latency of your application
- The wait time can be between 1 seconds to 20 seconds (20 seconds preferrable)
- Long Polling is preferrable to short Polling
- Long Polling can be enabled at the queue level or at the API level using WaitTimeSeconds

#### Types of Queues
| Feature          | Standard Queue                                                     | FIFO Queue                                                      |
|------------------|---------------------------------------------------------------------|-----------------------------------------------------------------|
|**Availability**|Available in all regions|Available in a few regions (US East, EU, Asia Pacific, etc.)|
|**Throughput**|Unlimited – Can handle a large number of transactions per second|High – Supports up to 3,000 messages per second with batching|
|**Delivery**|At least once – Messages might be delivered more than once|Exactly once – Each message is delivered only once|
|**Ordering**|Best effort – Messages may arrive out of order|Strict – Messages arrive in the exact order they are sent|
|**Use Case**|Ideal for high throughput applications|Ideal for applications where message order matters|

#### Tips for Auto Scaling
- When the number of EC2 instances (or other processor instances) increases, the transactions to database will be also increased.
SQS as a buffer to database writes is a good solution for this problem. Because SQS Queue is infinitely scalable.
This solution is only work if the client isn't impact by latency of writing to DB.

### SNS (Simple Notification Service)
#### Overview
- A web service follow pub-sub messaging paradigm that makes it easy to set up, operate, and send notifications from the cloud.
- SNS is used for sending a message to many receivers.
- The event producer only sends message to one SNS topic.
- As many event receivers as we want to receive the notifications from SNS topic.
- Each subscriber to the topic will get all the messages (not: new feature to filter messages).
- Up to 12,500,000 subscriptions per topic.
- 100,000 topics limit.
- Many AWS services can send data directly to SNS for notifications

#### Types
- FIFO: Destination is only SQS
- Standard: Destination could be email, SMS, Lambda, SQS, mobile application endpoints, HTTP, HTTPS.

#### How to publish
- Topic publish (using the SDK)
    - Create a topic
    - Create a subscription (or many)
    - Publish to the topic
- Direct Publish (for mobile apps SDK)
    - Create a platform application
    - Create a platform endpoint 
    - Publish to the platform endpoint
    - Works with GG GCM, Apple APNS, Amazon ADM

#### Security
##### Encryption
- In-flight encryption using HTTPS API
- At-rest encryption using KMS keys
- Client-side encryption if the client wants to perform encryption/decryption itself

##### Access Controls
IAM policies to regulate access to the SNS API
##### SNS Access Policies (similar to S3 bucket policies)

#### Fan Out pattern with SQS
- Push once in SNS, receive in all SQS queues that are subscriber
- Fully decoupled, no data loss
- SQS allows for : data persistence, delayed processing and retries of work
- Ability to add more SQS subscribers over time
- Cross-Region Delivery: works with SQS Queues in other regions.

##### Example Features
- Combine with S3 Event Notifications to listen to S3's events changes and fan-out all those events to multiple SQS
- SNS to Amazon S3 through Kinesis Data Firehose (Preprocess data) and then save the result to supported KDF's destinations
- Support FIFO as FIFO SQS as well: Ordring by Message group ID, deduplication using a deduplication ID
    - Can have SQS standard and FIFO queues as subscribers
- Message Filtering: Based on the message to deliver to the correct destination. (I.e deliver to subscription queue if the message contain subscription fields.)


### Kinesis
#### Overview
- Managed service for real-time processing of streaming data at scale
- Data is automatically replicated to 3 AZs
- Types of Kinesis Services:
  - Kinesis Data Streams: low-latency streaming ingest at scale
  - Kinesis Data Firehose: load streams into S3, Redshift, ElasticSearch & Splunk
  - Kinesis Data Analytics: analyze data streams with SQL or Apache Flink
  - Kinesis Video Streams: for streaming video content

#### Kinesis Data Streams
##### Components
- Producers: Applications sending data into streams. (i.e Applications, Client, SDK, Kinesis Agent).
(Data that shared the same partition goes to the same shard (ordering))
- Shards: Base throughput unit (for every shard)
  - 1MB/s or 1000 msg/seconds for incoming data
  - 2MB/s outgoing data per shard per consumer
  - Billed per shard hour
- Consumers: Applications reading and processing data from streams. (i.e Kinesis Client libraries, SDK, Lambda, Kinesis Data Firehose, Kinesis Data Analytics)

##### Retention
- Default: 24 hours
- Can be extended to 365 days

##### Capacity Modes
- Provisioned
  - Choose number of shards provisioned
  - Each shard gets 1MB/s in and 2MB/s out
  - Pay per shard hour
- On-demand
  - No need to provision/manage shards
  - Default capacity of 4MB/s or 4000 records/s
  - Scales automatically based on observed throughput peak during last 30 days
  - Pay per stream hour & data in/out
##### Security
- Control access/authorization using IAM policies
- Encryption in flight using HTTPS endpoints
- Endcryption at rest using KMS
- You can implement encryption/decryption of data on client side (harder)
- VPC Endpoints available for Kinesis to access within VPC.
- Monitor API calls using CloudTrail

#### Kinesis Data Firehose
- Fully managed service for delivering data streams: ETL with Lambda and save backup S3 objects
- Automatic scaling, serverless (with lambda)
- Supports transformation of data using Lambda
- Destinations:
  - AWS: S3, Redshift, ElasticSearch
  - 3rd party: Splunk, MongoDB, DataDog
  - Custom: HTTP Endpoint
- Pay for data going through Firehose

##### Security
- Control access using IAM policies
- Encryption in flight using HTTPS endpoints
- Encryption at rest using KMS
- Client side encryption possible
- VPC Endpoints available for Kinesis
- Monitor API calls using CloudTrail

##### Use Cases
- Application logs, metrics, IoT, clickstreams
- Real-time analytics
- Mobile data capture
- Gaming data feeds
- Complex Stream Processing

##### Comparison with SQS
- Kinesis
  - Data expires after retention period
  - Possibility to replay data
  - Meant for real-time big data, analytics and ETL
  - Ordering at the shard level
  - Enhanced fan-out capability
- SQS
  - Data deleted after consumption
  - No replay capability
  - Meant for decoupling applications
  - No ordering in standard queues
  - Individual message delay capability

##### Comparison Kinesis Data Streams vs Kinesis Data Firehose

| Feature | Kinesis Data Streams | Kinesis Data Firehose |
|---------|---------------------|----------------------|
| Purpose | Real-time streaming data ingestion and processing | Data delivery to destinations (ETL service) |
| Real-time | Real-time (sub-second) | Near real-time (minimum 60 seconds) |
| Scaling | Manual (provisioned) or automatic (on-demand) scaling with shards | Automatic scaling, fully managed |
| Processing | Custom processing using applications/Lambda | Built-in data transformation with Lambda |
| Retention | 24 hours to 365 days | No retention, immediate processing |
| Destinations | Custom consumers (EC2, Lambda, etc.) | S3, Redshift, ElasticSearch, 3rd party services |
| Management | Manage scaling, sharding, and consumers | Fully managed, serverless |
| Use Case | Real-time analytics, custom processing in streaming pipeline | Data ingestion into data lakes, warehouses |
| Fault tolerant| Support replay | Doesn't support replay capacity |

##### Comparison SQS vs SNS vs Kinesis Data Streams

| Feature | SQS | SNS | Kinesis Data Streams |
|---------|-----|-----|---------------------|
| Message Persistence | Messages deleted after consumption | No persistence | Configurable retention (1-365 days) |
| Delivery Model | Pull-based (polling) | Push-based (pub/sub) | Pull-based |
| Replay Capability | No replay | No replay | Supports replay |
| Message Ordering | No ordering (standard) / FIFO available | No ordering (standard) / FIFO available | Per-shard ordering |
| Throughput | Unlimited | Unlimited | Provisioned capacity with shards |
| Use Case | Decoupling applications | Fan-out messaging | Real-time data analytics and processing |
| Consumer Type | Single consumer (unless using fan-out) | Multiple subscribers | Multiple consumers |
| Scaling | Automatic | Automatic | Manual (shards) or on-demand |
| Message Processing | Individual messages | Broadcast to all subscribers | Stream processing |
| Data Transformation | No built-in transformation | No built-in transformation | Possible with Kinesis Analytics |

#### Problems
##### Ordering data into Kinesis
Sending using the Partition Key value. i.e `truck_id` (multiple truck_id)
Because the same key will always go to the same shard.

### Amazon MQ
#### Overview
- Amazon MQ is a managed message broker service that makes it easy to migrate to AWS and operate message brokers in the cloud
- Supports industry-standard APIs and protocols like JMS, NMS, AMQP, STOMP, MQTT, and WebSocket
- Managed Apache ActiveMQ or RabbitMQ engine
- Doesn't scale as much as SQS/SNS but supports traditional protocols
- Runs on servers, can run in Multi-AZ with failover
- Has both queue feature (~SQS) and topic features (~SNS)

#### Use Cases
- When you need to migrate applications using traditional message brokers to AWS without rewriting the messaging code
- Perfect for enterprise applications that need to migrate to the cloud
- Supports queue and pub/sub patterns

## Deployments

### ECS (Elastic Container Service)
#### Overview
- Amazon ECS is a fully managed container orchestration service that helps you run, stop and manage Docker containers on a cluster
- ECS is a key service for running containerized applications on AWS at scale
- ECS removes the need to install, operate, and scale your own cluster management infrastructure

#### Launch Types
- EC2 Launch Type
  - You manage the EC2 instances (infrastructure)
  - Each EC2 instance must run the ECS Agent to register in the ECS Cluster
  - AWS takes care of starting/stopping containers
  - More control over infrastructure, need to manage capacity
  
- Fargate Launch Type
  - Serverless - no EC2 instances to manage
  - You just create task definitions
  - AWS runs containers for you based on CPU/RAM you need
  - Pay for resources allocated to your containers
  - Simpler to use, less control, more expensive

#### Key Components
- Task Definition
  - Blueprint for your application (JSON file)
  - Contains container information: image, ports, memory, CPU requirements
  - Can specify multiple containers in a task definition
  
- ECS Service
  - Maintains specified number of tasks running at all times
  - Handles task placement and replacement if tasks fail
  - Can be integrated with Application Load Balancer
  
- ECS Cluster
  - Logical grouping of EC2 instances or Fargate tasks
  - Instances/tasks run the ECS agent
  - Can contain multiple services


#### Integration with other AWS Services
- Application Load Balancer (ALB) for load balancing
- ECR (Elastic Container Registry) for storing Docker images
- CloudWatch for monitoring and logging
- IAM for security and permissions
- VPC for networking

#### IAM Roles for ECS
- EC2 Instance Profile (EC2 Launch Type only)
  - Used by ECS agent
  - Makes API calls to ECS service
  - Pull Docker images from ECR
  - Reference sensitive data in Secrets Manager or SSM Parameter Store
  
- ECS Task Role
  - Allows each task to have specific role
  - Use different roles for different ECS Services
  - Task Role is defined in the task definition
  - Should be used for applications running in ECS

#### Launch Types Comparison

| Feature | EC2 Launch Type | Fargate Launch Type |
|---------|----------------|---------------------|
| Infrastructure Management | You manage EC2 instances | Fully managed by AWS |
| Cost | Cheaper for consistent workloads | Pay per task running |
| Control | Full control over instance types and configuration | Limited control, AWS manages infrastructure |
| Security Updates | You handle OS/infrastructure updates | AWS handles infrastructure updates |
| Scaling | Need to manage capacity and scaling of EC2 instances | Automatic scaling of tasks |
| Use Case | Large workloads with consistent usage | Variable workloads, need simplicity |
| Monitoring | Access to instance and container metrics | Container level metrics only |
| Maintenance | Higher maintenance overhead | Lower maintenance, fully managed |
| Network Mode | All networking modes supported | Only awsvpc network mode |
| Storage | Instance storage + EBS/EFS volumes | Ephemeral storage only + EFS = Serverless|

#### ECS Service Auto Scaling
##### Overview
- Automatically increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses AWS Application Auto Scaling
- ECS Service Auto Scaling (task level) ≠ EC2 Auto Scaling (instance level)
- Scaling based on:
  - ECS Service Average CPU Utilization
  - ECS Service Average Memory Utilization
  - ALB Request Count Per Target

##### Auto Scaling Types
- Target Tracking
  - Scale based on target value for a specific CloudWatch metric
  - Example: Keep average CPU utilization at 50%
  
- Step Scaling
  - Scale based on CloudWatch alarms
  - More granular control over scaling behavior
  
- Scheduled Scaling
  - Scale based on specified dates/times
  - Useful for predictable workload changes

##### EC2 Launch Type - Additional Scaling
- Auto Scaling Group Scaling
  - Scale EC2 instances based on CPU utilization
  - Add EC2 instances when running out of capacity
  
- Cluster Capacity Provider
  - Automatically provision and scale infrastructure for ECS tasks
  - Capacity Provider paired with an Auto Scaling Group
  - Add EC2 instances when you're missing capacity

##### Fargate Auto Scaling
- Simpler scaling as AWS manages infrastructure
- Based on task utilization metrics
- No need to manage EC2 instances
- Scaling policies:
  - Target tracking
  - Step scaling
  - Scheduled scaling

##### Service CPU/Memory Metrics
- CloudWatch metrics updated every minute
- Service CPU/Memory utilization
- Service CPU/Memory reservation
- Can be used for target tracking and step scaling

#### EventBridge Schedule to Invoke ECS Tasks
##### Overview
- EventBridge (CloudWatch Events) can be used to schedule ECS tasks
- Allows running tasks on a schedule without maintaining a continuously running service
- Useful for batch processing, maintenance tasks, or periodic workloads

##### Configuration
- Schedule Types
  - Fixed Rate: Run at specified intervals (every N minutes/hours/days)
  - Cron Expression: More complex scheduling patterns
  - One-time scheduled tasks

##### Components Required
- ECS Task Definition
  - Defines the container(s) to run
  - Must be registered and active
- ECS Cluster
  - Where the task will run
  - Can be EC2 or Fargate launch type
- IAM Roles
  - EventBridge needs permission to invoke ECS tasks
  - Task needs appropriate permissions for its workload

##### Use Cases
- Periodic Data Processing
  - Running ETL jobs
  - Data cleanup tasks
  - Report generation
- Maintenance Tasks
  - Database maintenance
  - Log rotation
  - System health checks
- Cost Optimization
  - Run tasks only when needed
  - No need for continuously running services

##### Best Practices
- Use Fargate for scheduled tasks to avoid managing infrastructure
- Set appropriate task timeouts
- Monitor task execution and completion
- Use task retries for reliability
- Implement proper error handling and notifications

### ECR (Elastic Container Registry)
#### Overview
- Fully managed Docker container registry service by AWS
- Store, manage, and deploy Docker container images
- Integrated with ECS and other AWS services
- Supports private and public repositories
- Highly available and scalable

#### Features
##### Storage
- Images stored in S3 behind the scenes
- Pay for storage used by container images
- Supports multiple versions of each image
- Lifecycle policies to manage image versions

##### Security
- IAM Authentication
  - Access control through IAM users/roles
  - ECR API integrated with IAM
  
- Image Scanning
  - Basic scanning (free)
    - Scans for CVEs (Common Vulnerabilities and Exposures)
    - One scan per image when pushed
  - Enhanced scanning (paid)
    - Continuous scanning
    - More detailed findings
    
- Encryption
  - Images encrypted at rest using KMS
  - Encrypted in transit (HTTPS)

##### Integration
- Works seamlessly with ECS and EKS
- Supports Docker CLI
- Cross-account access possible through IAM roles
- Supports cross-region replication

#### Image Lifecycle
##### Pushing Images
```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com

# Tag image
docker tag image:tag aws_account_id.dkr.ecr.region.amazonaws.com/repository:tag

# Push image
docker push aws_account_id.dkr.ecr.region.amazonaws.com/repository:tag
```

##### Pulling Images
```bash
# Pull image
docker pull aws_account_id.dkr.ecr.region.amazonaws.com/repository:tag
```

#### Lifecycle Policies
- Automatically clean up unused images
- Based on rules you define
- Examples:
  - Remove untagged images
  - Keep only N most recent images
  - Remove images older than X days

#### Best Practices
- Use image tagging strategy
  - Avoid using "latest" tag in production
  - Use semantic versioning
- Implement lifecycle policies
- Enable image scanning
- Use immutable tags
- Regularly clean up unused images
- Use cross-region replication for disaster recovery

#### Pricing
- Pay for:
  - Storage of images
  - Data transfer OUT
  - API requests
- No cost for:
  - Data transfer IN
  - Data transfer between ECR and ECS in same region

#### Use Cases
- Store container images for ECS/EKS deployments
- Share container images across AWS accounts
- Maintain private Docker registry
- Build CI/CD pipelines with container images


### EKS (Elastic Kubernetes Service)
#### Overview
- Managed Kubernetes service to run Kubernetes on AWS
- Kubernetes is an open-source system for automating containerized applications
- Alternative to ECS, similar goal but different API
- EKS runs upstream Kubernetes, can use all existing Kubernetes tools
- AWS manages the Kubernetes control plane
- Supports both EC2 and Fargate launch types

#### Components
##### Control Plane
- Managed by AWS
- Runs across multiple AZs for high availability
- AWS handles scaling and updates
- Automated version upgrades and patching
- Key Components:
  - API Server: Entry point for all REST commands
  - etcd: Distributed database storing cluster state
  - Controller Manager: Manages node/replication controllers
  - Scheduler: Assigns pods to nodes based on resource requirements

##### Worker Nodes
- EC2 instances that run your containers
- Can use managed node groups
- Can use self-managed nodes
- Can use Fargate (serverless)

##### Node Types
- Managed Node Groups
  - AWS creates and manages EC2 instances
  - Automatic updates and patching
  - Managed scaling
  - Uses Amazon EKS-optimized AMIs
  
- Self-Managed Nodes
  - You manage EC2 instances yourself
  - More flexibility and control
  - Need to handle updates and scaling
  
- Fargate
  - Serverless container deployment
  - AWS manages infrastructure
  - Pay per pod

##### How Components Work Together
1. Control Plane to Worker Communication
   - API Server acts as the hub for all communication
   - Worker nodes communicate with control plane via kubelet
   - All commands flow through the API server for security

2. Pod Scheduling Process
   - User submits pod deployment request to API Server
   - Scheduler identifies best node for pod placement
   - Selected node's kubelet creates the pod
   - Container runtime pulls and runs container images

3. Networking Flow
   - kube-proxy maintains network rules on nodes
   - Enables pod-to-pod communication across cluster
   - Implements services for pod load balancing

4. State Management
   - etcd stores cluster state and configuration
   - Controller Manager ensures desired state matches actual state
   - Continuous reconciliation loop maintains cluster health

#### Data Volumes
- Container Storage Interface (CSI) drivers
- Supports AWS EBS
- Supports Amazon EFS (works with Fargate)
- Supports Amazon FSx for Lustre
- Supports Amazon FSx for NetApp ONTAP

#### Networking
- Uses Amazon VPC networking
- Every pod gets an IP address
- Supports AWS VPC CNI (Container Network Interface)
- Can use alternative CNI plugins

#### Security
##### Authentication
- Uses IAM authentication
- Supports RBAC (Role-Based Access Control)
- Integration with AWS IAM Authenticator

##### Network Security
- Security Groups for worker nodes
- Network Policies for pod-to-pod communication
- Integration with AWS Shield and WAF

##### Pod Security
- Pod Security Policies
- Security Context
- Container Image Scanning

#### Load Balancing
- Supports AWS Application Load Balancer (ALB)
- Supports Network Load Balancer (NLB)
- Automatic integration through Kubernetes service objects

#### Monitoring and Logging
- Amazon CloudWatch Container Insights
- AWS CloudTrail for API logging
- Kubernetes native monitoring tools (Prometheus, etc.)
- Support for AWS X-Ray for tracing

#### Use Cases
- Microservices Architecture
- Hybrid Deployments
- Batch Processing
- Machine Learning
- Multi-cloud strategies (due to Kubernetes compatibility)

#### Comparison with ECS

| Feature | EKS | ECS |
|---------|-----|-----|
| Orchestration | Kubernetes | AWS proprietary |
| Learning Curve | Steeper (Kubernetes) | Easier (AWS-specific) |
| Control | More control, more complexity | Less control, simpler |
| Flexibility | More flexible, runs anywhere | AWS-specific |
| Pricing | More expensive (control plane cost) | No additional control plane cost |
| Tooling | All Kubernetes tools work | AWS-specific tools |
| Multi-cloud | Easier to move workloads | AWS-specific |
| Market Share | Popular for hybrid/multi-cloud | Popular for AWS-only |

#### Best Practices
- Use managed node groups when possible
- Implement proper security controls
- Use cluster autoscaling
- Regular updates and patches
- Proper monitoring and logging
- Use EKS-optimized AMIs
- Implement proper backup strategies

### App Runner
#### Overview
- Fully managed service that makes it easy to deploy web applications and APIs
- No infrastructure experience required
- Automatically builds and deploys the web application
- Automatic scaling, highly available
- VPC access support
- Connect to databases, caches, and message queue services

#### Source Code Support
##### Source Types
- Container Registry
  - Amazon ECR Public
  - Amazon ECR Private
  - Support for Dockerfiles
- Source Code Repository
  - GitHub
  - Support for Python, Node.js, Java, .NET Core

#### Features
##### Auto Scaling
- Automatic scaling based on requests
- Scale to zero when no requests (cost optimization)
- Configure min/max instances
- Concurrent request-based scaling

##### Networking
- Public endpoint by default
- VPC access (connect to RDS, ElastiCache, etc.)
- VPC security groups
- Custom domains with HTTPS

##### Security
- IAM integration
- Secrets/Parameter management
- Resource-based policies
- Encryption in transit

##### Monitoring
- Amazon CloudWatch integration
- Application logs
- Deployment and service metrics
- Health checks

#### Deployment
##### Configuration Options
- Memory and CPU configurations
- Environment variables
- Instance size selection
- Auto scaling configuration
- Health check settings

##### Deployment Options
- Rolling deployments
- Automatic rollback on failure
- Traffic splitting for testing
- Deployment triggers

#### Use Cases
- Web Applications
- APIs and Microservices
- Development and Test Environments
- Small to Medium Applications
- Proof of Concept Projects

#### Benefits
- Quick setup and deployment
- No infrastructure management
- Automatic scaling
- Cost-effective (pay for what you use)
- Built-in monitoring and logging
- Integrated security

#### Pricing
- Pay for compute and memory resources
- Additional charges for provisioned concurrency
- Data transfer costs apply
- No minimum fees
- Pay only for what you use

#### Limitations
- Region availability
- Container size limits
- Concurrent request limits
- Custom runtime limitations
- Network connectivity restrictions

### App2Container
#### Overview
- A command-line tool to help you migrate existing applications running on-premises or on EC2 to run in containers
- Supports Java and .NET applications
- Automatically containerizes and migrates applications to Amazon ECS or EKS

#### Features
- Analyzes applications and their dependencies
- Generates container images and deployment artifacts
- Creates ECS/EKS deployment configurations
- Integrates with existing CI/CD pipelines

#### Process
1. Install App2Container on the source server
2. Analyze applications
3. Extract and containerize applications
4. Deploy to AWS container services (ECS/EKS)

#### Benefits
- Simplifies containerization of existing applications
- Reduces manual effort in container migration
- Maintains application dependencies and configurations
- Generates container best practices
