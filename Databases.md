# DynamoDB
## Overview
- Fully managed NoSQL database
- Serverless with automatic scaling
- Single-digit millisecond performance
- Built-in security and backup
- Event-driven programming with DynamoDB Streams


## Key Features
### Tables, Items, and Attributes
- Tables: Collection of items
- Items: Collection of attributes (maximum size of an item is 400KB)
- Attributes: Fundamental data elements
- No fixed schema required
- Data types supported are:
    - Scala Types: String, Number, Binary, Bool, Null
    - Document Types: List, Map
    - Set Types - String Set, Number Set, Binary Set
Therefore, in DynamoDB you can rapidly evolve schemas

## RCU and WCU
RCU and WCU measures the number of reads and writes (respectively) per second.
RCU and WCU are decoupled, so you can increase/decrease each value separately.

### Primary Keys
- Partition Key (Hash Key)
  - Must be unique for each item
  - Used to distribute data across partitions
- Composite Key (Partition Key + Sort Key)
  - Allows multiple items with same partition key
  - Items sorted by sort key

### Secondary Indexes
- Global Secondary Index (GSI)
  - Different partition key than base table
  - Can be added/modified any time
- Local Secondary Index (LSI)
  - Same partition key as base table
  - Must be created at table creation

### Cross-region replication

## Capacity Modes
### Provisioned Mode
- Specify reads/writes per second
- Pay for provisioned capacity
- Can use auto-scaling
- Good for predictable workloads

### On-Demand Mode
- Pay-per-request pricing
- No capacity planning needed
- Good for unpredictable workloads
- More expensive for predictable workloads

## DynamoDB Streams
- Time-ordered sequence of item changes
- Stored for 24 hours
- Can trigger Lambda functions
- Use cases:
  - Replication in the same region and over regions
  - Analytics
  - Notifications
  - Audit trails

## Security
- Encryption at rest using KMS
- IAM policies for access control
- VPC endpoints available
- CloudWatch and CloudTrail integration

## Backup and Restore
### Point-in-time recovery
- Optionally enabled for the last 35 days
- Point-in-time recovery to any time withing the backup windown
- The recovery process creates a new table
### On-demand backups
- Full backups for long-term retention, until explicitely deleted
- Doesn't affect performance or latency
- Can be configured and managed in AWS Backup (enabled cross-region copy)
- The recovery process creates a new table
Zero impact on performance

## Integration with Amazon S3
### Export to S3 (must enable PITR)
- Works for any point of time in the last 35 days
- Doesn't affect the read capacity of your table
- Perform data analysis on top of DynamoDB
- Retain snapshots for auditing
- ETL on top of S3 data before importing back into DynamoDB
- Export in DynamoDB JSON or ION format
### Import from S3
- Import CSV, DynamoDB JSON or ION format
- Doesn't consume any write capacity
- Creates a new table
- Import errors are logged in CloudWatch Logs

### Use Cases
- Mobile/Web Applications
- Gaming Applications
- Session Management
- Digital Ledgers
- Real-time Analytics

## DynamoDB Accelerator (DAX)
### Overview
- Fully managed, highly available in-memory cache
- Microsecond latency for cached reads and queries
- Compatible with DynamoDB API calls
- No application logic modification required
- 5 minutes TTL for cache (default)

### Architecture
- Write-through caching service
- Caches the following data:
  - Item cache for GetItem and BatchGetItem operations
  - Query cache for Query and Scan operations
- Cluster consists of up to 10 nodes
- Primary node handles writes, replica nodes handle reads
- Multi-AZ capable for high availability

### Use Cases
- Ideal for read-intensive workloads
- Gaming leaderboards
- Session management
- Product catalogs
- Real-time bidding
- NOT suitable for:
  - Write-intensive workloads
  - Strongly consistent reads (DAX only supports eventually consistent reads)
  - Applications that don't require microsecond latency

## Cache Strategies
- Write-through: Data written to DynamoDB is automatically updated in DAX
- Lazy Loading: Data loaded into cache only when requested
- TTL configurable at cluster level

## Monitoring
- CloudWatch metrics available
- Monitor cache hits/misses
- CPU utilization
- Memory usage
- Network throughput

## Pricing
- Pay per node-hour
- No upfront costs
- Pricing based on node type selected

## TTL
- Automatically delete items after an expiry timestamp
- Use cases: reduce stored data by keeping only current items, adhere to redulartory obligations, web session handling,...
