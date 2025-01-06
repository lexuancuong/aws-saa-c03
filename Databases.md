# **DynamoDB Notes**

## **1. Overview**
- DynamoDB is a **fully managed NoSQL database** designed for high availability and scalability.
- **Serverless** architecture with automatic scaling for unpredictable workloads.
- **Single-digit millisecond performance** for most use cases.
- Features **built-in security**, **point-in-time recovery**, and **event-driven programming** with DynamoDB Streams.
- Ideal for modern applications requiring flexible schemas and real-time interactions.

### **Key Use Cases**
- Mobile and web applications.
- Real-time gaming leaderboards.
- IoT data storage and management.
- Digital ledgers.
- Real-time analytics.

---

## **2. Data Model**
### **Tables, Items, and Attributes**
- **Table**: Collection of items (rows).
- **Item**: Collection of attributes (columns), max size 400KB.
- **Attributes**: Fundamental data elements.
  - **No fixed schema** required, allowing rapid evolution.
  - **Data types**:
    - **Scalar**: String, Number, Binary, Boolean, Null.
    - **Document**: Map, List.
    - **Set**: String Set, Number Set, Binary Set.

### **Keys and Indexes**
#### **Primary Keys**
- **Partition Key (Hash Key)**: Used to distribute data across partitions. Must be unique.
- **Composite Key (Partition Key + Sort Key)**: Supports sorting multiple items under the same partition key.

#### **Secondary Indexes**
- **Global Secondary Index (GSI)**: Different partition key than the base table; can be added or modified later.
- **Local Secondary Index (LSI)**: Same partition key as the base table; must be created during table creation.

---

## **3. Performance and Scaling**
### **Read and Write Throughput**
- Provisioned using **Read Capacity Units (RCU)** and **Write Capacity Units (WCU)**.
  - RCU: 1 strongly consistent read per second for an item up to 4KB.
  - WCU: 1 write per second for an item up to 1KB.
- **Decoupled scaling** of RCU and WCU.

### **Capacity Modes**
#### **Provisioned Mode**
- Predefine capacity for predictable workloads.
- Supports auto-scaling based on traffic.

#### **On-Demand Mode**
- Pay-per-request pricing.
- Ideal for spiky or unpredictable workloads.
- Eliminates capacity planning.

### **Caching with DynamoDB Accelerator (DAX)**
- **Fully managed in-memory cache** for DynamoDB.
- Provides **microsecond latency** for reads.
- Write-through caching for synchronization with DynamoDB.
- **Not suitable for** write-heavy or strongly consistent read workloads.

---

## **4. DynamoDB Streams**
- **Event-driven feature** capturing a time-ordered sequence of changes.
- Stores data for **24 hours**.
- Triggers AWS Lambda functions for automation and integrations.
- **Use Cases**:
  - Multi-region replication.
  - Real-time analytics.
  - Notifications and audit trails.

---

## **5. Backup and Restore**
### **Point-in-Time Recovery (PITR)**
- Restore to any point within the last **35 days**.
- Recovery creates a new table.

### **On-Demand Backups**
- Create full backups for **long-term retention**.
- **No performance impact** during backup.
- Supports cross-region and cross-account copies.

### **Integration with Amazon S3**
#### **Export to S3**
- Must enable PITR.
- Export data for analytics, ETL, and auditing.
- Export in **DynamoDB JSON** or **ION format**.

#### **Import from S3**
- Supports **CSV**, **DynamoDB JSON**, or **ION format**.
- Creates a new table without consuming write capacity.

---

## **6. Security**
- **Encryption at rest** with AWS KMS.
- **IAM policies** for fine-grained access control.
- **VPC endpoints** for secure, private communication.
- **CloudWatch** and **CloudTrail** for monitoring and auditing.

---

## **7. Monitoring and Troubleshooting**
- **CloudWatch Metrics**:
  - Table and partition-level metrics.
  - DAX metrics: cache hit/miss rates, CPU, memory.
- **Alerts**:
  - Set alarms for capacity usage or throttling events.
- **CloudTrail Logs** for audit trails and API call tracking.

---

## **8. Advanced Features**
### **Time-to-Live (TTL)**
- Automatically deletes expired items based on an attribute's timestamp.
- **Use Cases**:
  - Session management.
  - Regulatory compliance.
  - Reducing stored data by retaining only current items.

### **Cross-Region Replication**
- Enables multi-region disaster recovery and high availability.

---

## **9. Pricing**
- Pay based on:
  - Read/write requests (on-demand mode).
  - Provisioned capacity (provisioned mode).
  - Additional costs for features like Streams, backups, and DAX nodes.

---

## **10. Exam Tips**
- Understand **RCU/WCU calculations** and scenarios.
- Familiarize yourself with **use cases for GSIs and LSIs**.
- Be clear on **DynamoDB Streams** and **event-driven architectures**.
- Know when to use **provisioned vs. on-demand capacity**.
- Remember the role of **DAX for caching** and its limitations.

# **Amazon RDS**

## **1. Overview**
- **Amazon Relational Database Service (RDS)** is a fully managed relational database service.
- Supports popular database engines:
  - **Amazon Aurora**, **MySQL**, **PostgreSQL**, **MariaDB**, **Oracle**, **Microsoft SQL Server**.
- Handles database management tasks like **provisioning**, **patching**, **backups**, and **scaling**.
- Provides high availability with **Multi-AZ Deployments** and **automatic failover**.
- Suitable for applications requiring structured data and complex queries.

### **Key Use Cases**
- Transactional workloads (OLTP).
- Web and mobile applications.
- E-commerce platforms.
- Data warehousing (with Aurora or RDS Proxy).
- ERP, CRM, and business applications.

---

## **2. RDS Database Engines**
### **Amazon Aurora**
- **High performance**, proprietary database engine with MySQL and PostgreSQL compatibility.
- Up to **5x faster** than standard MySQL, 3x faster than PostgreSQL.
- **Continuous backups to S3** and replication across multiple Availability Zones.
- Serverless and global database options available.

### **MySQL and MariaDB**
- Popular open-source engines for general-purpose databases.
- Community and Amazon-provided versions supported.

### **PostgreSQL**
- Advanced open-source engine with support for complex queries, JSON, and extensibility.

### **Oracle and SQL Server**
- Commercial engines for enterprise-grade applications.
- License-included or BYOL (Bring Your Own License) options.

---

## **3. High Availability and Durability**
### **Multi-AZ Deployments**
- Ensures high availability with standby instances in a different AZ.
- Automatic failover in case of instance failure.
- Synchronous replication ensures data consistency.

### **Read Replicas**
- Asynchronous replication for read-heavy workloads.
- Available for MySQL, PostgreSQL, MariaDB, and Aurora.
- Supports up to 5 replicas (Aurora supports 15).
- Can be promoted to standalone databases.

---

## **4. Scaling**
### **Vertical Scaling**
- Increase instance size (CPU, memory, storage).
- Requires downtime during resizing.

### **Horizontal Scaling**
- Add **Read Replicas** for distributing read traffic.
- Aurora supports automatic scaling with **Aurora Serverless**.

### **Storage Scaling**
- Supports **automatic storage scaling** up to specified limits.
- Aurora automatically scales storage up to 128TB.

---

## **5. Backup and Restore**
### **Automated Backups**
- Enabled by default; stored in S3.
- Retention period: 1 to 35 days.
- Supports **point-in-time recovery**.

### **Manual Snapshots**
- User-initiated full database backups.
- Retained indefinitely until explicitly deleted.
- Can be shared across AWS accounts and copied across regions.

---

## **6. Security**
- **Encryption**:
  - At rest using AWS KMS.
  - In-transit with SSL/TLS.
- **Network Security**:
  - Deploy in a **VPC** for isolated networking.
  - Control access using **Security Groups**.
- **IAM Integration**:
  - Fine-grained access control for database operations.
- **Database Authentication**:
  - Native database authentication.
  - **IAM-based authentication** (MySQL and PostgreSQL).
  - **Kerberos** for Oracle and SQL Server.

---

## **7. Monitoring and Maintenance**
### **Monitoring Tools**
- **CloudWatch**: Monitor CPU, memory, IOPS, storage, and more.
- **Enhanced Monitoring**: OS-level metrics.
- **Performance Insights**: Analyze query performance and database load.

### **Maintenance**
- Automatic patching for database engine and OS.
- Maintenance windows can be customized.

---

## **8. Pricing**
- **Pay-as-you-go** model based on instance hours, storage, and data transfer.
- Factors influencing cost:
  - Database engine and instance type.
  - Storage type (SSD or magnetic).
  - Multi-AZ and read replica configurations.
  - Backup storage and data transfer.

---

## **9. Advanced Features**
### **RDS Proxy**
- **Fully managed database proxy** for improving performance and scalability.
- Reduces connection management overhead.
- Supported for MySQL, PostgreSQL, and Aurora.
- Ideal for **serverless architectures** like AWS Lambda.

### **Aurora Features**
- **Global Database**:
  - Replicate across multiple regions.
  - Enables fast disaster recovery and low-latency global reads.
- **Aurora Serverless**:
  - On-demand automatic scaling based on application load.
  - Cost-effective for intermittent workloads.

---

## **10. Exam Tips**
- Understand the differences between **Multi-AZ** and **Read Replicas**.
- Familiarize with **RDS Proxy** and its use cases.
- Know the features and benefits of **Aurora** compared to other engines.
- Be clear on **security configurations** (encryption, IAM, SSL/TLS).
- Learn how **automated backups** and **snapshots** work for recovery.
- Understand cost implications of **Multi-AZ**, **read replicas**, and **storage scaling**.

## Tips
- Amazon RDS Custom for Oracle facilitates customizations to the underlying Oracle database as well as OS with minimum infrastructure maintenance effort.

---

# **Amazon Aurora**

## **1. Overview**
- **Fully managed** relational database engine compatible with MySQL and PostgreSQL
- Up to **5x performance** of MySQL and **3x** of PostgreSQL
- **Distributed**, fault-tolerant, self-healing storage system
- **Auto-scaling** from 10GB to 128TB
- **Serverless** option available for variable workloads
- Designed for **99.99% availability**

### **Key Use Cases**
- Enterprise applications requiring high availability
- SaaS applications with multi-tenant databases
- Gaming applications with large, rapidly growing datasets
- Applications requiring read scaling
- Mission-critical applications needing automatic failover

---

## **2. Architecture**
### **Storage Architecture**
- Data stored in **6 copies** across **3 Availability Zones**
- Storage automatically grows in **10GB increments**
- **Self-healing**: continuous data block scanning and repair
- Separation of **compute** and **storage** layers

### **Endpoints**
- **Cluster Endpoint**: Primary instance for write operations
- **Reader Endpoint**: Load-balanced read operations
- **Custom Endpoints**: User-defined endpoint for specific instance sets
- **Instance Endpoints**: Direct connection to specific instances

---

## **3. High Availability and Replication**
### **Aurora Replicas**
- Up to **15 Aurora Replicas**
- **Sub-10 second failover**
- Support for **cross-region replicas**
- **Zero-downtime** patching
- **Automatic failover** to replica with highest priority

### **Global Database**
- Spans multiple AWS Regions
- **< 1 second** replication lag
- Supports **disaster recovery**
- Enables **global read scaling**
- **RPO of 1 second**, RTO of < 1 minute

---

## **4. Scaling**
### **Aurora Serverless**
- **Auto-scaling** based on actual usage
- Pay only for resources consumed
- Scales compute capacity in **seconds**
- Ideal for **variable workloads**
- **Cost-effective** for intermittent usage

### **Read Scaling**
- Add up to **15 Aurora Replicas**
- **Reader endpoint** for automatic load balancing
- **Custom endpoints** for workload optimization
- **Cross-region read replicas** available

---

## **5. Backup and Recovery**
### **Continuous Backup**
- **Automatic backups** to Amazon S3
- Retain backups for up to **35 days**
- **Point-in-time recovery** to any second
- **Zero impact** on database performance

### **Snapshots**
- **Manual snapshots** stored indefinitely
- Can be shared across AWS accounts
- **Automated snapshots** with custom retention
- **Cloning** feature for quick database copies

---

## **6. Security**
### **Network Security**
- Runs in **VPC**
- **Security Group** control
- **IAM authentication**
- **Private subnet** deployment option

### **Encryption**
- **At-rest encryption** using KMS
- **In-transit encryption** using SSL/TLS
- **Automated key rotation**
- **Encrypted snapshots**

---

## **7. Monitoring and Performance**
### **Performance Insights**
- **Real-time** performance monitoring
- **SQL query** analysis
- **Load trending** and analysis
- **No additional cost**

### **Enhanced Monitoring**
- **OS metrics** in real-time
- **50+ metrics** available
- **1-second** to **60-second** intervals
- Integration with **CloudWatch**

---

## **8. Cost Optimization**
### **Pricing Components**
- **Instance hours**
- **I/O operations**
- **Storage consumption**
- **Data transfer**
- **Backup storage**

### **Cost-Saving Features**
- **Serverless** for variable workloads
- **Auto-scaling** storage
- **Reserved instances** for predictable workloads
- **Stop/Start** feature for development environments

---

## **9. Advanced Features**
### **Aurora Serverless v2**
- **Fine-grained** auto-scaling
- **Instant** scaling
- **Cost-effective** for variable workloads
- Compatible with other Aurora features

### **Parallel Query**
- Pushes computation to storage layer
- Improves analytical query performance
- Available for Aurora MySQL

### **Multi-Master**
- **Multiple write nodes**
- Continuous availability
- Immediate failover
- Ideal for write-scaling

---

## **10. Exam Tips**
- Understand **Aurora Serverless** use cases
- Know **Global Database** features and benefits
- Remember **high availability** architecture (6 copies, 3 AZs)
- Understand difference between **endpoint types**
- Know **backup** and **recovery** options
- Be familiar with **scaling** capabilities
- Understand **security** features and encryption options
- Know when to choose Aurora over standard RDS

## Comparison between RDS and Aurora
| Feature | Amazon RDS | Amazon Aurora |
|---------|------------|---------------|
| **Performance** | Standard database performance | Up to 5x MySQL and 3x PostgreSQL performance |
| **Storage** | Max 16TB per database | Auto-scales up to 128TB |
| **Replication** | Up to 5 read replicas | Up to 15 read replicas |
| **Availability** | 99.95% (Multi-AZ) | 99.99% |
| **Failover Time** | 1-2 minutes | < 30 seconds |
| **Backup** | Manual snapshots and automated backups | Continuous backup to S3 |
| **Storage Replication** | 2 copies (Multi-AZ) | 6 copies across 3 AZs |
| **Scaling** | Manual vertical and horizontal | Automatic storage scaling, Aurora Serverless option |
| **Global Database** | Not available | Available with < 1 second replication lag |
| **Cost** | Pay per instance | Pay for consumed resources, more cost-effective at scale |
| **Recovery Point** | Up to last 5 minutes | Up to last second |
| **Engine Support** | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server | MySQL and PostgreSQL compatible only |
| **Serverless Option** | No | Yes (Aurora Serverless) |
| **Cloning** | Snapshot and restore | Fast cloning feature |
| **Connection Management** | Standard | Advanced with RDS Proxy |

---

# **Amazon ElastiCache**

## **1. Overview**
### **Core Concepts**
- Fully managed **in-memory caching service**
- Supports two engines:
  - **Redis**: Advanced data structures, persistence, replication
  - **Memcached**: Simple key-value store, multi-threading
- Microsecond latency for real-time applications
- Reduces database load for read-heavy workloads

### **Key Use Cases**
- **Session Management**: Store session data for stateless servers
- **Database Query Caching**: Cache frequent DB queries
- **Real-time Analytics**: Gaming leaderboards, real-time dashboards
- **API Rate Limiting**: Implement request rate limiting
- **Real-time Bidding**: Ad-tech and gaming applications

---

## **2. Redis vs. Memcached (Exam Focus)**
### **Choose Redis When You Need**
- **Persistence**: Data survives node failures
- **Complex Data Types**: Lists, Sets, Sorted Sets, Hashes
- **High Availability**: Multi-AZ with automatic failover
- **Backup/Restore**: Point-in-time recovery
- **Geospatial Capabilities**: Location-based features
- **Pub/Sub Messaging**: Event-driven architectures
- **Transactions**: Multiple operations atomically

### **Choose Memcached When You Need**
- **Simple Object Caching**: Key-value only
- **Multi-threading**: Utilize multiple CPU cores
- **Horizontal Scaling**: Add/remove nodes easily
- **Lowest Latency**: Simpler architecture
- **Partition Data**: Across multiple nodes
- **No Special Features**: Basic caching only

---

## **3. High Availability and Fault Tolerance**
### **Redis**
- **Multi-AZ** with automatic failover
- **Primary** and **Replica** nodes
- **Cluster Mode**: 
  - Up to 500 nodes
  - Data partitioned across shards
  - Scale reads and writes

### **Memcached**
- **No Built-in Replication**
- **Auto Discovery**: Nodes automatically join/leave cluster
- **Node Failure**: Data lost on node failure

---

## **4. Performance Optimization**
### **Caching Strategies**
- **Lazy Loading**: Cache on read
  - Pros: Only cache what's needed
  - Cons: Cache miss penalty
- **Write Through**: Update cache on write
  - Pros: Data always fresh
  - Cons: Write penalty
- **TTL (Time To Live)**:
  - Prevent stale data
  - Balance freshness vs. hit rate

### **Common Patterns**
- **Cache-Aside**: Application manages cache
- **Read-Through**: Cache manages DB reads
- **Write-Through**: Cache manages DB writes
- **Write-Behind**: Async DB updates

---

## **5. Security**
### **Network Security**
- **VPC** deployment
- **Security Groups**
- **Subnet Groups**
- **In-transit encryption** (Redis)

### **Authentication**
- **Redis AUTH**
- **IAM Authentication** (Redis)
- **Access Control Lists** (Redis)

### **Encryption**
- **At-rest encryption** (Redis)
- **In-transit encryption** (Redis)
- **SSL/TLS support** (Redis)

---

## **6. Monitoring and Maintenance**
### **Key Metrics**
- **CPU Utilization**
- **Memory Usage**
- **Cache Hits/Misses**
- **Evictions**
- **Connections**

### **CloudWatch Integration**
- **Host-level metrics**
- **Engine-specific metrics**
- **Custom alarms**

---

## **7. Scaling**
### **Redis Scaling**
- **Vertical**: Change node type
- **Horizontal**: Add/remove replicas
- **Cluster Mode**: Add/remove shards
- **Online Resharding**

### **Memcached Scaling**
- **Vertical**: Change node type
- **Horizontal**: Add/remove nodes
- **Auto Discovery**

---

## **8. Cost Optimization**
### **Pricing Factors**
- **Node Type**
- **Number of Nodes**
- **Data Transfer**
- **Reserved Nodes** (up to 60% savings)

### **Best Practices**
- **Right-size** node types
- Use **reserved nodes** for steady workloads
- **Monitor** usage patterns
- **Clean up** unused resources

---

## **9. Common Architectures**
### **Web Applications**
- **Session Management**
- **API Caching**
- **Database Query Caching**

### **Real-time Applications**
- **Gaming Leaderboards**
- **Real-time Analytics**
- **Live Dashboards**

### **Microservices**
- **API Rate Limiting**
- **Distributed Locking**
- **Message Broker**

---

## **10. SAA-C03 Exam Tips**
### **Key Decision Points**
- When to choose **Redis vs. Memcached**
- Understanding **Multi-AZ** requirements
- **Scaling** options and limitations
- **Security** requirements and features

### **Common Scenarios**
- **Session Management**: Redis for persistence
- **Gaming/Leaderboards**: Redis sorted sets
- **Simple Caching**: Memcached for basic needs
- **High Availability**: Redis Multi-AZ

### **Remember**
- Redis is **feature-rich** but more complex
- Memcached is **simple** but limited
- **Encryption** options vary by engine
- **Backup/Restore** only available in Redis
- **Cost implications** of different architectures

# **Additional AWS Database Services**

## **Amazon DocumentDB**
- **MongoDB-compatible** document database service
- It is used to store, query and index JSON data.
- Key points:
  - Fully managed, scalable, highly available
  - Auto-scales storage from 10GB to 64TB
  - Up to 15 read replicas
  - Automatically scales to workloads with milions of requests per seconds
  - Use when: Need MongoDB compatibility with AWS integration, complex JSON queries, analytics

## **Amazon Neptune**
- **Graph database** service
- Key points:
  - Supports Property Graph and RDF
  - Use for: Social networks, recommendation engines, fraud detection
  - Up to 15 read replicas over 3 AZ
  - Automatic failover in < 30 seconds
  - If the dataset has many relationships (e.g., friendships, likes) and queries frequently involve traversing these relationships.
  - Graph databases like Neptune are specifically optimized for such scenarios. The query would involve traversing the graph to find "friends of AI", their videos, and then aggregating likes.

### **Neptune Streams**
- Real-time ordered sequence of every change to your graph data
- Key features:
  - Changes available immediately after writing
  - No duplicates, strict ordering guaranteed
  - Accessible through HTTP REST API
- **Use cases**:
  - Send notifications for specific graph changes
  - Synchronize graph data with other stores (S3, OpenSearch, ElastiCache)
  - Cross-region data replication in Neptune

## **Amazon Keyspaces**
- **Apache Cassandra-compatible** service
- Key points:
  - Serverless, pay-per-use
  - Use for: High-scale industrial apps, IoT data
  - No servers to manage
  - Automatic scaling

## **Amazon QLDB**
- **Quantum Ledger Database**
- Key points:
  - Immutable, cryptographically verifiable transaction log
  - Use for: Financial records, supply chain, medical records
  - 2-3x better performance than blockchain frameworks
  - Serverless

## **Amazon Timestream**
- **Time-series database** service
- Key points:
  - Purpose-built for IoT and operational applications
  - Automatic scaling
  - Built-in time series analytics
  - Use for: IoT data, DevOps monitoring, industrial telemetry

---
