# **Amazon Redshift**

## **1. Key Concepts**
- **Fully managed** petabyte-scale **data warehouse** service
- Based on **PostgreSQL**, optimized for **OLAP** (Online Analytical Processing)
- **Columnar storage** for efficient querying of large datasets
- **Massively Parallel Processing (MPP)** architecture
- **10x better performance** than traditional data warehouses
- Costs **~1/10th** of traditional data warehousing solutions

## **2. Architecture & Components**
### **Cluster Structure**
- **Leader Node**: 
  - Query planning and result aggregation
  - Coordinates parallel execution
  - Handles external communication
- **Compute Nodes**:
  - Execute queries in parallel
  - Store data in slices
  - Scale from 1 to 128 nodes

### **Node Types**
- **RA3 Nodes** (Recommended):
  - Managed storage, separate compute from storage
  - Automatic data placement
  - Hot data in SSD, cold in S3
- **DC2 Nodes**:
  - Compute-intensive workloads
  - SSD storage
- **DS2 Nodes**:
  - Legacy, large storage capacity
  - HDD storage

## Backup
- Can be backup to S3 manually or automated: every 8 hours, every 5GB or on a schedule. Set retentions
- Can configure Amzon Redshift to automatically copy snapshots (automated or manual) of a cluster to another AWS Region.

## Data Source
- Kinesis Firehose
- S3 (copy command)
- EC2 storage

## **3. Key Features for SAA-C03**
### **Redshift Spectrum**
- Query data directly in S3 without loading
- Extends queries to your data lake
- Separates storage from compute
- Cost-effective for infrequently accessed data

### **Concurrency Scaling**
- Automatically adds cluster capacity
- Handles concurrent queries
- Pay per second for additional clusters
- Free credits available each day

### **AQUA (Advanced Query Accelerator)**
- Hardware-accelerated cache
- Improves query performance
- Automatically available for RA3 nodes

## **4. Integration Points (Exam Focus)**
### **Data Loading**
- **S3**: COPY command (most efficient)
- **DynamoDB**: Direct import
- **Database Migration Service**: Real-time replication
- **Kinesis Firehose**: Real-time streaming
- **EC2/EMR**: ETL processing

### **Data Access**
- **QuickSight**: Business intelligence
- **Athena**: Query both Redshift and S3
- **EMR**: Big data processing
- **SageMaker**: Machine learning

## **5. Security & Compliance**
- **VPC** deployment
- **IAM** integration
- **KMS** encryption at rest
- **CloudWatch** monitoring
- **CloudTrail** auditing
- **Multi-AZ** for RA3 nodes (newer feature)

## **6. Common Exam Scenarios**
### **When to Choose Redshift**
- **Complex queries** on large datasets
- Need for **joins** and **aggregations**
- **Predictable query patterns**
- **Regular ETL workloads**
- **Business intelligence** requirements

### **When Not to Choose Redshift**
- **Small datasets** (< 100GB)
- **OLTP** workloads (use RDS instead)
- **Unpredictable/ad-hoc** queries (consider Athena)
- **Real-time** data ingestion (use Kinesis first)

## **7. Cost Optimization Tips**
- Use **Reserved Instances** for predictable workloads
- Leverage **Spectrum** for infrequently accessed data
- Enable **auto-suspend** for dev/test clusters
- Monitor and adjust **concurrency scaling**
- Choose appropriate **node types** based on workload

---

# **Amazon Athena**

## **1. Core Concepts**
- **Serverless** query service to analyze data in S3
- Uses standard **SQL** (built on Presto)
- Pay **only for data scanned**: 5$ per TB of scanned data
- No infrastructure to manage
- Supports various file formats (CSV, JSON, ORC, Parquet, Avro)

## **2. Key Features for SAA-C03**
### **Performance & Cost Optimization**
- Use **columnar formats** (Parquet/ORC) for best performance
- Partition data for reduced scan costs
- Compress data to reduce costs
- Use workgroups to control query costs

### **Integration Points**
- **S3**: Primary data source
- **Glue Data Catalog**: Schema management
- **QuickSight**: Visualization
- **Lambda**: Queries from applications
- **CloudWatch**: Query monitoring

## **3. Common Use Cases**
- **Ad-hoc queries** on S3 data lakes
- **Log analysis** (VPC Flow Logs, CloudTrail, ALB logs)
- **One-time data transformations**
- **Cost-effective BI** on occasional queries
- **Federated queries** to various data sources

### **Performance & Cost Optimization**
- Use **columnar formats** (Parquet/ORC) for best performance
  - Significantly reduces data scanning costs
  - Use AWS Glue to convert data to Parquet/ORC
  - Huge performance improvement over row-based formats

- **Compression Options**:
  - Supported: bzip2, gzip, lz4, snappy, zlib, zstd
  - Reduces data retrieval size
  - Lower scanning costs

- **Partitioning Strategy**:
  - Structure: `s3://bucket/table_path/<partition_column>=<value>/`
  - Example: `s3://athena-examples/flight/parquet/year=1991/month=1/day=1/`
  - Creates virtual columns for efficient filtering
  - Dramatically reduces data scanning

- **File Optimization**:
  - Use larger files (> 128 MB)
  - Minimizes overhead from multiple small files
  - Better query performance

- **Additional Tips**:
  - Use workgroups to control query costs
  - Implement data lifecycle policies
  - Regular monitoring of scan patterns

## **4. When to Choose Athena**
### **Best For**
- **Serverless** analytics needs
- **Irregular/unpredictable** query patterns
- **Quick insights** from S3 data
- **Pay-per-query** model preferred
- **Simple setup** requirements

### **Not Suitable For**
- **Complex ETL** workflows (use EMR/Glue)
- **Real-time queries** (use Kinesis)
- **Frequent, complex joins** (use Redshift)
- **ACID transactions** (use RDS/Redshift)

## **5. Exam Tips**
- Remember: **Query S3 data directly**
- Cost based on **data scanned**
- Use **partitioning** and **compression** for optimization
- Perfect for **occasional, ad-hoc queries**
- Integrates well with **AWS analytics stack**

---

# **Amazon OpenSearch**

## **1. Core Concepts**
- **Fully managed** search and analytics engine
- Available in two modes:
  - Managed cluster
  - Serverless cluster
- Enables **full-text search** capabilities
- Complements other AWS databases

## **2. Key Features**
### **Search Capabilities**
- Search across **any field**
- Support for **partial matches**
- **Flexible querying** (unlike DynamoDB's primary key limitation)
- Can be enabled with SQL support via plugin

### **Data Ingestion**
- Multiple ingestion sources:
  - Kinesis Data Firehose
  - AWS IoT
  - CloudWatch Logs
  - Custom applications

### **Security**
- **Authentication**: 
  - Amazon Cognito integration
  - IAM-based access control
- **Encryption**:
  - KMS encryption at rest
  - TLS encryption in transit
- **Network**: 
  - VPC support
  - Fine-grained access control

## **3. Common Use Cases**
- **Log Analytics**
- **Full-text search** for applications
- **Real-time application monitoring**
- **Security analytics**
- **Clickstream analytics**

## **4. Integration Patterns**
### **Common Architectures**
- DynamoDB + OpenSearch for flexible queries
- CloudWatch Logs + OpenSearch for log analytics
- S3 + OpenSearch for document search
- Custom apps + OpenSearch for search functionality

## **5. Exam Tips**
- Remember: **Complement to other databases**
- Perfect for **search-intensive workloads**
- Use when you need to **search across any field**
- Consider for **partial text matching** requirements
- Understand difference between **managed vs serverless** options

---

# Kinesis
## Overview
- Managed service for real-time processing of streaming data at scale
- Data is automatically replicated to 3 AZs
- Types of Kinesis Services:
  - Kinesis Data Streams: low-latency streaming ingest at scale
  - Kinesis Data Firehose: load streams into S3, Redshift, ElasticSearch & Splunk
  - Kinesis Data Analytics: analyze data streams with SQL or Apache Flink
  - Kinesis Video Streams: for streaming video content

## Kinesis Data Streams
Amazon Kinesis Data Streams (KDS) is a massively scalable and durable real-time data streaming service.
It can continuously capture gigabytes of data per second from hundreds of sources such as website clickstreams, database event streams, financial transactions, social media feeds, IT logs, and location-tracking events.
### Components
- Producers: Applications sending data into streams. (i.e Applications, Client, SDK, Kinesis Agent).
(Data that shared the same partition goes to the same shard (ordering))
- Shards: Base throughput unit (for every shard)
  - 1MB/s or 1000 msg/seconds for incoming data
  - 2MB/s outgoing data per shard per consumer
  - Billed per shard hour
- Consumers: Applications reading and processing data from streams. (i.e Kinesis Client libraries, SDK, Lambda, Kinesis Data Firehose, Kinesis Data Analytics)

### Retention
- Default: 24 hours
- Can be extended to 365 days

### Capacity Modes
- Provisioned
  - Choose number of shards provisioned
  - Each shard gets 1MB/s in and 2MB/s out
  - Pay per shard hour
- On-demand
  - No need to provision/manage shards
  - Default capacity of 4MB/s or 4000 records/s
  - Scales automatically based on observed throughput peak during last 30 days
  - Pay per stream hour & data in/out
### Security
- Control access/authorization using IAM policies
- Encryption in flight using HTTPS endpoints
- Endcryption at rest using KMS
- You can implement encryption/decryption of data on client side (harder)
- VPC Endpoints available for Kinesis to access within VPC.
- Monitor API calls using CloudTrail

## Kinesis Data Firehose
- Fully managed service for delivering data streams: ETL with Lambda and save backup S3 objects
- Automatic scaling, serverless (with lambda)
- Supports transformation of data using Lambda
- Destinations:
  - AWS: S3, Redshift, ElasticSearch
  - 3rd party: Splunk, MongoDB, DataDog
  - Custom: HTTP Endpoint
- Pay for data going through Firehose

### Security
- Control access using IAM policies
- Encryption in flight using HTTPS endpoints
- Encryption at rest using KMS
- Client side encryption possible
- VPC Endpoints available for Kinesis
- Monitor API calls using CloudTrail

### Use Cases
- Application logs, metrics, IoT, clickstreams
- Real-time analytics
- Mobile data capture
- Gaming data feeds
- Complex Stream Processing

### Comparison with SQS
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

### Comparison Kinesis Data Streams vs Kinesis Data Firehose

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

### Comparison SQS vs SNS vs Kinesis Data Streams

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

## Problems
### Ordering data into Kinesis
Sending using the Partition Key value. i.e `truck_id` (multiple truck_id)
Because the same key will always go to the same shard.

