# Disaster Recovery
## Overview
- Any event that has a negative impact on a company's business continuty or finances is a disaster.
- Disaster recovery (DR) is about preparing for and recovering from a disaster.
- What kind of disaster recovery?
    - On-premise => On-premise: traditional DR, and very expensive
    - On-premise => AWS Cloud: hybrid recovery
    - AWS Cloud Region A => AWS Cloud Region B

## Key Concepts
- **Recovery Time Objective (RTO):** The maximum allowable time to restore operations after a disruption.
- **Recovery Point Objective (RPO):** The maximum allowable data loss measured in time.

## Disaster Recovery Strategies

Disaster Recovery (DR) involves preparing for and recovering from unplanned outages that could impact business operations.
AWS offers multiple strategies to ensure high availability and fault tolerance based on the Recovery Time Objective (RTO) and Recovery Point Objective (RPO).

### 1. Backup and Restore
- **Description:** (ONLY) DATA is periodically backed up to a secure location and restored during a disaster.
- **RTO:** Hours to days.
- **RPO:** Minutes to hours.
- **Implementation:**
  - Use **Amazon S3** and **S3 Glacier** for cost-effective backups.
  - Automate backups using AWS Backup or **Lifecycle Policies**.
  - Use **AWS Elastic Disaster Recovery** for restoring EC2 instances.

---

### 2. Pilot Light
- **Description:** A MINIMAL VERSION of the production environment is maintained, containing core infrastructure elements. During a disaster, the environment is scaled up.
- **RTO:** Minutes to hours.
- **RPO:** Minutes.
- **Implementation:**
  - Keep critical databases and core services always running in a secondary region.
  - Use **AWS Auto Scaling** and **Elastic Load Balancers** to scale resources.
  - Replicate data with **Amazon RDS Multi-AZ** or **DynamoDB Global Tables**.

---

### 3. Warm Standby
- **Description:** A SCALED-DOWN BUT FUNCTIONAL REPLICA of the production environment is maintained in another region. It can be scaled up to full capacity when needed.
- **RTO:** Minutes.
- **RPO:** Seconds to minutes.
- **Implementation:**
  - Deploy critical applications in a secondary region using **AWS CloudFormation**.
  - Use **Route 53** for DNS failover routing.
  - Synchronize databases with **Amazon Aurora Global Database** or **DynamoDB Streams**.

---

### 4. Multi-Site (Active-Active)
- **Description:** FULL PRODUCTION ENVIRONMENTS run simultaneously in two or more regions. Traffic is distributed using load balancing or DNS.
- **RTO:** Near zero.
- **RPO:** Near zero.
- **Implementation:**
  - Use **AWS Global Accelerator** or **Route 53** for global traffic routing.
  - Deploy applications in multiple regions with **Amazon ECS/EKS** or **Elastic Beanstalk**.
  - Maintain real-time data replication with **DynamoDB Global Tables** or **S3 Cross-Region Replication**.

---
## Practical Implementation Tips

### Backup Strategies
- **AWS Native Backups:**
  - EBS Snapshots for block storage
  - RDS automated backups and snapshots for databases
  - Regular data pushes to S3 with Intelligent-Tiering / Glacier using Lifecycle Policies
  - S3 Cross-Region Replication for geographic redundancy
- **On-Premise to AWS Migration:**
  - Use AWS Snowball for large-scale data transfer
  - Implement Storage Gateway for hybrid cloud storage

### High Availability Components
- **Network and DNS:**
  - Route 53 for cross-region DNS failover
  - Site-to-Site VPN as backup for Direct Connect
- **Database and Storage:**
  - RDS Multi-AZ deployments
  - ElastiCache Multi-AZ for in-memory caching
  - Amazon EFS for scalable file storage
  - S3 for highly available object storage

### Data Replication Solutions
- **Database Replication:**
  - RDS Cross-Region Replication
  - Aurora Global Databases for global reach
  - Database migration from on-premise to RDS
- **Storage Solutions:**
  - Storage Gateway for hybrid storage integration
  - S3 Cross-Region Replication

### Automation and Recovery
- **Infrastructure Automation:**
  - CloudFormation templates for environment recreation
  - Elastic Beanstalk for application deployment
- **Monitoring and Recovery:**
  - CloudWatch alarms for EC2 instance recovery/reboot
  - Lambda functions for custom automation workflows

### Chaos Engineering
- **Resilience Testing:**
  - Implement chaos engineering principles (like Netflix's "simian-army")
  - Regularly terminate random EC2 instances to test recovery
  - Use AWS Fault Injection Simulator for controlled chaos experiments

### Keep Balance
Choosing the right DR strategy depends on your business requirements, particularly RTO and RPO.
AWS provides flexible services and tools to implement efficient and cost-effective disaster recovery solutions.


## Best Practices for Disaster Recovery
1. **Automate:** Use Infrastructure as Code (IaC) tools like **AWS CloudFormation** or **Terraform** for consistency and quick deployment.
2. **Test Regularly:** Perform regular DR drills using AWS **Fault Injection Simulator** or other testing tools.
3. **Monitor and Alert:** Use **Amazon CloudWatch**, **AWS Trusted Advisor**, and **AWS Health Dashboard** for proactive monitoring.
4. **Secure Data:** Encrypt backups with **AWS KMS** and ensure appropriate IAM permissions.

# AWS Database Migration Service (DMS)

## Overview

AWS Database Migration Service (DMS) is a managed service that simplifies transferring databases to AWS.
It supports various source and target databases, including commercial and open-source options.

## Key Features
* **Wide Compatibility:** Migrates between a broad range of database engines.
* **Minimal Downtime:** Offers methods to minimize disruption during the migration process.
* **Scalability and Availability:** Handles large-scale migrations with high availability.
* **Data Integrity:** Ensures data consistency and accuracy throughout the migration.
* **Cost-Effectiveness:** Pay-as-you-go pricing based on usage.
* **Security:** Encrypts data both in transit and at rest.
* **Multi-AZ Enabled**: DMS provisions and maintains a synchronously stand replica in a different AZ.

## Use Cases
* **Lift and Shift:** Migrate existing databases to AWS without major application changes.
* **Database Upgrades:** Move to newer versions of database engines on AWS.
* **Data Warehousing:** Migrate data for analytics and reporting purposes.
* **Disaster Recovery:** Replicate databases to AWS for business continuity.
* **Cloud Migration:** Migrate on-premises databases as part of a cloud migration strategy.

## Implementation Continous Replication*
1. **Create a Replication Instance:** A compute engine (EC2) that runs the DMS software.
2. **Create a Migration Task:** Define source and target databases, specify settings (e.g., schema conversion, data filtering), and schedule the migration.
3. **Run the Migration:** DMS initiates the migration process, replicating data.
4. **Validate and Cutoff:** After completion, validate data in the target database and switch traffic to the new system.

## DMS Sources and Targets
### Sources
- On-premises and EC2 instances databases
- Azue
- Amazon RDS: all including Aurora
- Amazon S3
- Document DB

### Targets
- On-premises and EC2 instances databaes
- RDS
- Redshift, DynamoDB, S3
- OpenSearch Service
- Kinesis Data Streams
- Apache Kafka
- DocumentDB and Amazon Neptune
- Redis and Babelfish

## AWS Schema Conversion Tool (SCT)
- Covert your database's schema from one engine to another:
    - Example OLTP: to MySQL, PostgresSQL, Aurora
    - Example OLAP: to Amazon Redshift
- Can install in the on-premises' server.
- Do not need to use SCT for migration if you are migrating the same DB engine

## Common Database Migration
### RDS MySQl/PostgresSQL -> Aurora
Option 1: Dump database and restore as the mainDB source for Aurora DB
=> Downtime
Option 2: Create an Aurora Read Replica from RDS MySQL and when the replication lag is 0, promote it as its own DB cluster.
=> Take time and cost $

### External MySql/PostgresSQL -> Aurora
Option 1: 
- Use Percona XtraBackup to create a file backup in Amazon S3
- Create an Aurora MySQL DB from Amazon S3

Option 2:
- Create an Aurora MySQL DB
- Dump the data and restore in Aurora (Slower than S3 method)
**Use DMS if both databases are up and running**

# AWS Backup

## Overview

AWS Backup is a fully managed service that simplifies and automates backups across a wide range of AWS services, including Amazon EC2, Amazon RDS, Amazon EBS, and more. 
It centralizes and automates the backup process, providing a single pane of glass for managing and monitoring backups across your AWS environment.
AWS Backup is a crucial service for protecting your data and ensuring business continuity in the AWS cloud.
By centralizing and automating backups, you can significantly reduce the risk of data loss and improve your overall data protection posture.

## Supported services
- EC2/EBS
- S3
- RDS / Aurora / DynamoDB
- DocumentDB / Neptune
- EFS / FSx
- Storage Gateway

## Key Features                                                            
* **Automated Backups:** Schedule and automate regular backups according to your defined policies.
* **Centralized Management:** Manage and monitor backups for various AWS services from a single console.
* **Centralized Storage:** Stores backups in AWS, such as Amazon S3, for durability and cost-effectiveness.
* **Cross-Region Replication:** Replicate backups to different AWS regions for enhanced disaster recovery.
* **Restore Flexibility:** Easily restore individual files, databases, or entire applications from backups.
* **Compliance:** Helps meet compliance requirements by providing audit trails and reporting.
* **Cost-Effective:** Pay-as-you-go pricing based on backup storage and retrieval.
* **Vault Lock** Enforce a WORM (Write Once Read Many): Even the root user cannot delete backups when enabled.

## Use Cases

* **Data Protection:** Protect critical data from accidental deletion, corruption, and disasters.
* **Disaster Recovery:** Enable rapid recovery of applications and data in case of failures.
* **Compliance:** Meet regulatory and industry compliance requirements for data protection.
* **Development and Testing:** Create and restore backups for development and testing purposes.
* **Archiving:** Archive data for long-term storage and retrieval.

## How it Works
1. **Create Backup Plans:** Define backup rules within backup plans, specifying resources to be backed up, schedules, retention policies, and encryption settings.
2. **Schedule Backups:** AWS Backup automatically schedules and executes backups according to the defined plans.
3. **Store Backups:** Backups are stored securely and durably in AWS S3.
4. **Restore Data (When needed):** Easily restore individual files, databases, or entire applications from backups.

### Key Concepts

* **Backup Plan:** A collection of rules that define how and when to back up specific resources.
* **Backup Rule:** Specifies the resources to be backed up, the schedule, retention settings, and encryption options.
* **Backup Vault:** A container for storing backups within AWS S3.
* **Recovery Point Objective (RPO):** The maximum amount of data that could be lost in the event of a failure.
* **Recovery Time Objective (RTO):** The time it takes to recover systems and applications after a failure.

# Other Services
## AWS Application Discovery Service 
AWS Application Discovery Service helps you assess your on-premises data center by automatically collecting information about servers, **applications**, and dependencies, enabling streamlined migration planning.
It provides detailed insights to facilitate workload migration to AWS.
Resulting data can be viewed within AWS Migration Hub.

## Application Migration Service
AWS SMS enables incrementally automated migration of on-premises servers to AWS by orchestrating large-scale server migrations efficiently.
It supports incremental replication, reducing downtime and simplifying the migration process.

# Common Problems
## Transferring large amount of data into AWS
Option 1: Over the internet / Site to Site VPN
- Immediate to setup
- Take a long time
Over Direct Connect 1Gbps:
- Long time setup (over a month)
- Take less time (10x faster)
Over Snowball:
- Will take 2 to 3 snowballs in parallel
- Take about 1 week for the end to end transfer
- Can be combined with DMS

## Manage AWS Cloud with VMware
- Supported by VMware vSphere-based private, public and hybrid cloud environments.
