# S3
## Overview
- Amazon S3 is one of the main building blocks of AWS

## Use cases
- Backup and Storage
- Disaster Recovery
- Archive
- Hybrid Cloud Storage
- Application hosting
- Media hosting
- Data lakes and big data analytics
- Software delivery
- Static website

## Components
### Buckets
- As root directories
- Must have a globally unique name (across all regions all accounts)
- Buckets are defined at the region level

### Objects
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

## Security
### User-Based
- IAM Policies - which API calls should be allowed for a specific user from IAM

### Resource-Based
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

### Note
- an IAM principal can access an S3 object if the user IAM permissions allow it or the resource policy allows it
- Could encrypt objects in Amazon S3 using encryption keys
- Configure Bucket Policy Handons
  - use awspolicygen to generate the JSON template to configure S3
  - If you get 403 forbidden error, ensure that the bucket policy allows public reads

## Versioning
### Scope
- It is enabled at the bucket level

### Reasons
- Protect against unintended deletes
- Easy rollback to previous version

### Notes
- Any file that is not versioned prior to enabling versioning will have version "null"
- Suspending versioning does not delete the previous versions

## Replication
- Must enable Versioning in source and destination buckets

### Types
- Cross Region Replication (CRR)
  - Compliance, lower latency access replication across accounts
- Same Region Replication (SRR)
  - Log aggregation, live replication between production and test accounts

- Buckets can be in different AWS accounts
- Copying is asynchronous
- Must give proper IAM permissions to S3

### Notes
- After enabling Replication, only new objects are replicated
  - You can replicate existing objects using S3 Batch Replication (for failed replications as well)
- Can replicate delete markers from source to target
- Deletions with a version ID are not replicated (to avoid malicious deletes)
- There is no chaining of replication

## Storage Classes
### Standard - General Purpose
- 99.99% Availability
- Used for frequently accessed data
- Low latency and high throughput
- Sustain 2 concurrent facility failures
- Use cases: Big data analytics, mobile & gaming applications, content distribution...

### Infrequent Access
- Used for less frequently accessed but required rapid access when needed
- Lower cost than S3 standard
- S3 Standard - Infrequent Access
  - 99.9% availability
  - Use cases: Disaster recovery, backups
- S3 One Zone - Infrequent Access
  - High durability 99.9999999% in a single AZ, data lost when AZ is destroyed
  - 99.5% availability
  - Minimum duration is 30 days before you can transit objects from S3 standard to S3 one-zone.
  - Use cases: Storing secondary backup copies of on-premise data or data you can recreate

### Glacier
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
- NOTES: YOU DON’T HAVE TO CONFIGURE THE S3 LIFECYCLE POLICY AND WAIT FOR 30 DAYS TO MOVE THE DATA TO **GLACIER DEEP ARCHIVE**. Other storage classes still require.



### S3 Intelligent Tiering
- Small monthly monitoring and auto-tiering fee
- Move objects automatically between Access Tier based on usage
- There are no retrieval charges in S3 intelligent tiering

### Lifecycle Rules
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

### Notes:
- Can move between classes manually or using S3 Lifecycle configurations

## S3 Requester Pays
- In general, bucket owners pay for all Amazon S3 storage and data transfer cost associated with their bucket
- With requester pays buckets, the requester instead of the bucket owner pays the cost of the request and the data download from the bucket
- Helpful when you want to share large datasets with other accounts
- The requester must be authenticated to AWS

## S3 Event Notifications
### Type
- S3:ObjectsCreated
- S3:ObjectRemoved
- Object name filtering possible (*.jpg)
- Use case: generate thumbnails of images uploaded to S3
- Can create as many S3 events as desired
- Asynchronous events
- Usually go to Amazon EventBridge, Amazon EventBridge supports delivering all messages to over 18 AWS services as destinations.
- Can also configure to send S3 events to SNS, SQS and lambda
  - Need to set up destination's policy to allow S3 notification send message to

## Baseline Performance
- Amazon S3 automatically scales to high request rates (latency 100-200ms)
- Your application can achieve at least 3500 PUT/COPY/POST/DELETE or 5,500 GET/HEAD requests per second per prefix in a bucket
- There are no limits to the number of prefixes in a bucket
- Example (object path => prefix)
  - bucket/prefix/foldername/filename
- If you spread reads across all four prefixes evenly, you can achieve 22,000 requests per second for GET and HEAD

### Multi-Part upload
- recommended for files > 100MB, must use for files > 5GB
- Can help parallelize uploads (speed up transfers)

### Transfer Acceleration
- Increase transfer speed by transferring file to an AWS edge location which can forward quickly the data to the S3 bucket in the target region
- Compatible with multi-part upload

### Byte-Range Fetches
- Parallelize GETs by requesting specific byte ranges
- Better resilience in case of failures
- => Can be used to speed up downloads

## Batch Operations
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

## Storage Lens
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

## Object Encryption
### Server-Side Encryption (SSE)
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

### Client-side encryption
- Use client libraries such as Amazon S3 Client-Side encryption library
- Clients must encrypt data themselves before sending to Amazon S3
- Clients must decrypt data themselves when retrieving from Amazon S3
- Customer fully manages the keys and encryption cycle

### Notes
- Bucket policies are evaluated before default encryption
- Could force encryption using a bucket policy and refuse any API call to PUT an S3 object without encryption headers (SSE-KMS or SSE-C)

## S3 CORS
- If a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers
- CORS issues usually happen because the objects could be stored in different S3 buckets (=> different URLs)
- Need to add Access-Control-Allow-Origin to allow this cross buckets access
  - Configuring involves AllowedHeaders, AllowedMethods and AllowedOrigins, ExposeHeaders, MaxAgeSeconds

## Amazon S3 - MFA Delete
### Definition
- force users to generate a code on a device before doing important operations on S3

- MFA will be required to permanently delete an object version
- Suspend versioning on the bucket
- MFA won't be required to
  - Enable Versioning
  - List deleted versions
- Only the bucket owner(root account) can enable/disable MFA Delete

## Access Logs
- For audit purpose, you may want to log all access to S3 buckets
- Any request made to S3, from any account authorized or denied will be logged into another S3 bucket
- That data can be analyzed using data analysis tools...
- The target logging bucket must be in the same AWS region

### Warnings
- Do not set your logging bucket to be the monitored bucket
- It will create a logging loop and your bucket will grow exponentially

## Pre-signed URLs
- Generate pre-signed URLs using S3 console, AWS CLI or SDK
- Users given a pre-signed URL inherit the permissions of the user that generated the URL for GET/PUT
- Use cases
  - Allow only logged-in users to download a premium video from your S3 bucket
  - Allow an ever-changing list of users to download files by generating URLs dynamically
  - Allow temporarily a user to upload a file to a precise location in your S3 bucket

## Glacier Vault Lock
- Adopt a WORM (Write Once Read Many) model
- Create a Vault Lock Policy
- Lock the policy for future edits (can no longer be changed or deleted)
- Helpful for compliance and data retention

## S3 Object Lock
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

## Access Points
- Manage the permissions of group of people or departments
- Configure via Policy setup
- Each access point has its own DNS name
- an access point policy (similar to bucket policy) - manage security at scale
- We can define the access point to be accessible only from within the VPC
- You must create a VPC Endpoint to access the Access Point
- The VPC Endpoint Policy must allow access to the target bucket and Access Point

## S3 Object Lambda
- Use AWS Lambda Functions to change the object before it is retrieved by the caller application
- Only one S3 bucket is needed, on top of which we create S3 Access Point and S3 Object Lambda Access Points
- Use cases
  - Converting data format
  - Resizing and watermarking images on the fly using caller-specific details, such as the user who requested the object

## S3 Bucket Policy
- Bucket policies in Amazon S3 can be used to add or deny permissions across some or all of the objects within a single bucket.
- Policies can be attached to users, groups, or Amazon S3 buckets, enabling centralized management of permissions.
- With bucket policies, you can grant users within your AWS Account or other AWS Accounts access to your Amazon S3 resources.
- You can further restrict access to specific resources based on certain conditions. For example, you can restrict access based on request time (Date Condition), whether the request was sent using SSL (Boolean Conditions), a requester’s IP address (IP Address Condition), or based on the requester's client application (String Conditions). To identify these conditions, you use policy keys.

## Notes
- Amazon S3 Storage class analyzes storage access patterns to help you decide when to transition the right data to the right storage class. Storage class analysis does not give recommendations for transitions to the ONEZONE_IA or S3 Glacier storage classes.

# EBS
## Overview
- Network USB Stick
- EBS could attach to 1 EC2 instance at 1 time
- Could configure to terminate together with EC2 or not
- One EC2 instance could attach multiple EBS
- Must be in the same AZ with EC2

## EBS Snapshots
- Make a backup of EBS volume at a point in time
- Not necessary to detach volume to do snapshot, but recommend
- Can copy snapshot across AZ or Region
- EBS Snapshot Archive Tier is 75% cheaper
- Take within 24 to 72 hours for restoring the archive
- Recycle bin
  - Can setup retention 1 day -> 1 year
  - Fast snapshot restore
  - Force to install all from snapshot

## EBS Volume Types
- gp2/gp3 (SSD)
  - General purpose SSD volume that balances price and performance for a wide variety of workloads (Can be root volume)
- io1/io2 (SSD)
  - Highest performance SSD for mission critical low latency or high through workloads (can be root volume)
- st1 (HDD)
  - low cost HDD volume design for frequently accessed throughput intensive workloads
- sc1 (HDD)
  - lowest cost HDD volume designed for less frequently accessed workloads

# EFS
## Overview
- Network file system for Linux instances, POSIX filesystem

# EC2 Instance Store
- EBS volumes are network drives with good but limited performance
- If you need a high performance hardware disk, use EC2 instance store

## Use cases
- Significantly better for I/O performance
- EC2 Instance Store lose their storage if they are stopped
- Good for buffer scale/scratch data/ temporary content
- Risk of data loss if hardware fails
- Backup and Replication are your responsibility

# FSx
## Overview
- Launch 3rd party high-performance file systems on AWS
- Fully managed service

## Types
### Windows File Server
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

### Lustre
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

### NetApp ONTAP
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

### OpenZFS
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

# Snowball
## Overview
- A offline service that allows users to transfer large amounts of data to their AWS cloud, even in areas with limited or no network connectivity. It's useful for businesses that collect data in remote locations, such as those in the architecture construction, energy and gaming industries

## Types
- Snowcone
  - 8 TB HDD -> 14 TB SSD
- Snowball Edge
  - 80TB-210TB

## Edge computing
- Process data while it's being created on an edge location

## How it works
1. Order the device via AWS management console
2. Receive and copy data
3. Ship back
4. Data Import by AWS

## Variants
- Snowball Edge Storage Optimized
  - Primarily for bulk data transfer and storage
- Snowball Edge Compute Optimized
  - Include powerful compute capability for edge processing
- Snowcone
  - A smaller, more portable device for small-scale data transfers edge computing

# AWS Storage Gateway
## Overview
- Hybrid cloud storage that bridges your on-premises infrastructure with AWS storage services

## Use cases
- Enables you to use AWS cloud storage for backups, archiving, and data tiering while keeping your on-premises applications running seamlessly

## Types
- File Gateway
  - Provides an interface for applications to access files via standard protocols like NFS or SMB
  - Data is stored in Amazon S3 as objects
- Tape Gateway
  - Virtual tape library for backup applications
  - Data is stored in S3 and archived to S3 Glacier or Glacier Deep Archive
- Volume Gateway
  - Cached mode: Frequently accessed data is cached locally with all data stored in S3
  - Store mode: Entire dataset is stored on-premises, with backups sent to EBS

# AWS Data Sync
- Move large amount of data to and from on-premises/other cloud to AWS (need agents). Or AWS to AWS (different storage services) - no agent needed.
- Can synchronize to S3 (any storage class - including Glacier), Amazon EFS
- Replication task can be scheduled hourly, daily or weekly
- File permissions and metadata are preserved (NFS POSIX, SMB,...)
- Ideal for datasets up to 100TB (bigger ones should be handled by Snowball).
