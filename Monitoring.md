# AWS CloudWatch - Monitoring & Observability

## Overview
Amazon CloudWatch is AWS's primary monitoring and observability service that provides:
- Real-time monitoring of AWS resources and applications
- Collection and tracking of metrics, logs, and events
- Automated actions and alerting
- Dashboards and visualizations
- Cross-account and cross-region monitoring

## Core Components

### 1. CloudWatch Metrics
#### Default Metrics
- Automatically collected from AWS services
- Basic Monitoring:
  - Free tier
  - 5-minute intervals
  - Default metrics (CPU, disk, network)
- Detailed Monitoring:
  - Paid feature
  - 1-minute intervals
  - Enhanced granularity
  - Better for Auto Scaling

#### Custom Metrics
- User-defined metrics published via API/SDK
- Resolution options:
  - Standard: 1-minute granularity
  - High Resolution: 1-second granularity
- Supports dimensions for metric segmentation
- Common uses:
  - Application metrics
  - Business metrics
  - Operations metrics

### 2. CloudWatch Logs
#### Components
- Log Groups:
  - Container for log streams
  - Shared retention and permissions
  - Metric filter application point
- Log Streams:
  - Sequence of log events
  - Same source (instance/application)
  - Chronologically ordered

#### Features
- Real-time log monitoring
- Configurable retention (1 day - never expire)
- Log Insights for SQL-like queries
- Export capabilities:
  - S3 export: Log data can take up to 12 hours to become available for export
  - Kinesis Data Firehose
  - Lambda subscriptions
  - Open Search
- Encryption using KMS (default)
- Cross-account subscription: send log events to resources in a different AWS account (KDS, KDF)
- Logs Aggregation of multi-account and multi regions

#### CloudWatch Logs Insights
##### Overview
- Interactive log analytics service
- Query multiple log groups across accounts/regions
- Purpose-built query language (similar to SQL)
- Real-time and historical analysis
- Automatic field discovery
- Visualization capabilities

##### Key Features
- **Query Language**:
  - Filter and pattern matching
  - Regular expressions
  - Mathematical operations
  - Statistical functions
  - Time-based analysis
  - Limit and sort results

- **Sample Queries**:
  ```
  # Find the 25 most recent error events
  fields @timestamp, @message
  | filter @message like /error/
  | sort @timestamp desc
  | limit 25
  ```

### 3. CloudWatch Alarms
Alarms are used to trigger notifications for any metric.
#### States
- OK: Metric within threshold
- ALARM: Metric breached threshold
- INSUFFICIENT_DATA: Not enough data

#### Period
- Lengh of time seconds to evaluate the metric
- High resolution custom metrics: 10 sec, 30 sec or multiples of 60 sec

#### Actions
Can trigger:
- Auto Scaling actions
- EC2 actions (stop, terminate, reboot or recover)
- SNS notifications
- Systems Manager actions
- Lambda functions

#### Types
- Metric Alarms:
  - Static thresholds
  - Anomaly detection
  - Math expressions
- Composite Alarms:
  - Multiple conditions (combining of multiple alarms)
  - AND/OR logic
  - Reduce alarm noise

### 4.EventBridge (CloudWatch Events)
#### Overview
- Serverless event bus service
- Delivers a real-time stream of system events
- Connects application data from your own apps, SaaS, and AWS services
- Simplifies event-driven architectures
- Supports multiple event buses per account
- Ability to replay archived events
- Ability to filter specific events
- Automatically archive events that flow through EventBridge. This helps you retain a history of events for later analysis, replay, or debugging.

#### Event Types
1. **AWS Service Events**
   - Generated automatically by AWS services
   - Examples:
     - EC2 instance state changes
     - AWS Console sign-in events
     - S3 bucket changes
     - RDS snapshots completion
     - AWS Health events

2. **Scheduled Events**
   - Cron expressions: `cron(0 8 ? * MON-FRI *)`
   - Rate expressions: `rate(5 minutes)`
   - Common use cases:
     - Daily data cleanup
     - Weekly report generation
     - Periodic health checks
     - Automated backups
     - Resource scheduling (start/stop)

3. **Partner Events**
   - SaaS partner integrations
   - Examples: Zendesk, DataDog, Auth0
   - Automatically ingest events from supported partners

4. **Custom Events**
   - Your application's custom events
   - JSON format
   - Can include custom fields and metadata

#### Common Use Cases
1. **Application Integration**
   - Microservices communication
   - Webhook processing
   - API Gateway integration
   - Cross-account event routing

2. **Automated Operations**
   - Resource provisioning
   - Configuration updates
   - Backup automation
   - Log processing

3. **Security Automation**
   - Security group changes
   - IAM policy modifications
   - Unauthorized API calls
   - GuardDuty findings

4. **Cost Optimization**
   - Resource scheduling
   - Development environment management
   - Unused resource cleanup
   - Budget alerts

#### Target Types & Integration
1. **Compute**
   - Lambda functions
   - ECS tasks
   - Step Functions
   - Batch jobs

2. **Messaging**
   - SNS topics
   - SQS queues
   - Kinesis streams
   - Event buses (cross-account)

3. **Management**
   - Systems Manager automation
   - EC2 actions (stop/start)
   - API Gateway endpoints
   - CloudWatch Logs

#### Best Practices
1. **Event Pattern Design**
   - Use specific patterns
   - Include relevant fields only
   - Test patterns thoroughly
   - Document pattern logic

2. **Error Handling**
   - Configure dead-letter queues
   - Monitor failed events
   - Set up retry policies
   - Implement error notifications

3. **Performance**
   - Use event filtering
   - Batch events when possible
   - Consider event bus quotas
   - Monitor event latency

4. **Security**
   - Use IAM roles and policies
   - Encrypt sensitive data
   - Implement least privilege
   - Audit event patterns regularly

#### Limits and Quotas
- Events per second: 10,000 (default)
- Event size: 256 KB max
- Targets per rule: 5 (default)
- Rules per bus: 300 (default)
- API request quota: 50 TPS

## CloudWatch Insights
CloudWatch Insights is a powerful feature of Amazon CloudWatch 
that allows you to perform real-time interactive log queries on your AWS application and infrastructure logs. 
It is designed to help developers, DevOps engineers, and system administrators efficiently analyze and debug issues in their applications or systems by querying log data stored in Amazon CloudWatch Logs.

### CloudWatch Application Insights
#### Overview
- Automated monitoring setup for applications and related AWS services
- Helps detect and diagnose problems across application stack

#### Key Features
- Automatic resource discovery and monitoring setup
- Intelligent problem detection using ML
- Automated dashboards for visualization
- Integration with Systems Manager OpsCenter
- Correlation of metrics, logs, and events
- Root cause analysis suggestions

#### Data Management
- Problem retention: 55 days
- Observations retention: 60 days
- Supports KMS encryption

### CloudWatch Container Insights
#### Overview
- Monitoring and diagnostics for containerized applications
- Supports ECS, EKS, Kubernetes on EC2, and Fargate
- Collects metrics at cluster, node, pod, and container levels

#### Key Features
- Automated metrics collection and aggregation
- Performance log events in embedded metric format
- Automatic dashboard creation
- Container-level diagnostic information
- Failure detection and alerting

#### Implementation
- Uses containerized CloudWatch agent
- Supports cross-account monitoring
- KMS encryption support (symmetric keys only)
- Charged per observation (enhanced) or as custom metrics (original)

### CloudWatch Lambda Insights
#### Overview
- Monitoring and troubleshooting solution for serverless applications
- Collects, aggregates, and summarizes system-level metrics
- Provides automated dashboards for Lambda functions
- Available for both x86 and ARM architectures

### Features
- Single-function and multi-function views
- Automatic anomaly detection
- Real-time monitoring
- Integration with CloudWatch alarms
- Cross-account monitoring support

### CloudWatch Contributor Insights
#### Overview
- Analyzes log data to identify top contributors
- Creates time-series data showing top-N contributors
- Built-in and custom rules for analysis
- Real-time monitoring of who/what is impacting system performance

#### Implementation
- Rules define how to process and analyze logs
- Supports CloudWatch Logs and S3 as data sources
- Results available within seconds
- Integration with CloudWatch dashboards
- Supports rule creation via console or API

## Important Service-Specific Metrics

### EC2
#### Integrations
- By default, no logs from your EC2 machine will go to CloudWatch.
- You need to run a CloudWatch agent on EC2 to push the log files you want.
- Make sure IAM permissions are correct.
- CloudWatch log agent can be setup on-premises too.
- CloudWatch Logs Agent
    - Old version of Agent
    - Can only send to CloudWatch Logs
- CloudWatch Unified Agent
    - Collect additional system-level metrics such as RAM, processes, etc,...
    - Collect logs to send to CloudWatch Logs
    - Centralized configuration using SSM Parameter Store
    - Collected directly on your Linux server/EC2 instance
#### Metrics
- CPUUtilization (%)
- DiskReadOps/DiskWriteOps
- NetworkIn/NetworkOut
- StatusCheckFailed
- Netstat (number of TCP and UDP connections, net packets, bytes)
- Memory utilization (requires CloudWatch agent)
- Disk space (requires CloudWatch agent)
- Swap Space (free, used, used %)

### RDS
- CPUUtilization
- DatabaseConnections
- FreeStorageSpace
- ReadIOPS/WriteIOPS
- ReadLatency/WriteLatency
- FreeableMemory

### ELB
- RequestCount
- HealthyHostCount/UnHealthyHostCount
- Latency/TargetResponseTime
- SurgeQueueLength
- SpilloverCount
- 5XX/4XX errors

### Lambda
- Invocations
- Duration
- Errors
- Throttles
- ConcurrentExecutions
- Iterator age (for stream invocations)

## CloudWatch Agent
### Purpose
- Collect metrics and logs from EC2 and on-premises
- Enhanced metrics collection
- Custom metrics support

### Types
- Standard CloudWatch Agent:
  - Metrics and logs collection
  - Configurable collection intervals
  - Custom metrics support
- CloudWatch Logs Agent (older):
  - Logs only
  - Being replaced by standard agent

### Collected Metrics
- Memory utilization
- Disk space usage
- Disk I/O
- Network utilization
- Process information
- Custom metrics

## Best Practices
1. **Monitoring Strategy**
   - Use detailed monitoring for critical resources
   - Set appropriate retention periods
   - Implement proper tagging
   - Use metric filters effectively

2. **Alarm Configuration**
   - Set meaningful thresholds
   - Use composite alarms for complex scenarios
   - Configure proper evaluation periods
   - Implement escalation procedures

3. **Log Management**
   - Define retention policies
   - Use log aggregation
   - Implement structured logging
   - Regular log analysis

4. **Cost Optimization**
   - Monitor CloudWatch costs
   - Use basic monitoring where appropriate
   - Set proper retention periods
   - Clean up unused resources

## Security
- IAM roles and policies for access
- KMS encryption for logs
- VPC endpoints for private access
- CloudTrail integration for API logging
- Cross-account sharing controls

## Integration Points
- Auto Scaling Groups
- SNS/SQS
- Lambda
- Systems Manager
- EventBridge
- S3
- Kinesis
- Third-party tools

## Exam Tips
1. **Know the Basics**
   - Metrics intervals (basic vs detailed)
   - Alarm states
   - Log retention options
   - Event routing capabilities

2. **Remember Integration Points**
   - How services work together
   - Common architectures
   - Auto Scaling integration
   - Notification options

3. **Understand Limitations**
   - Metric resolution options
   - Log retention limits
   - API quotas
   - Cross-region capabilities

4. **Cost Considerations**
   - Basic vs detailed monitoring
   - Custom metrics pricing
   - Log storage costs
   - API request charges

---

# AWS CloudTrail

## Overview
- AWS CloudTrail is a service that enables governance, compliance, and operational and risk auditing of your AWS account (for every AWS account - useful for managing many staff in an organization)
- It records account activity and API calls made in your AWS environment, providing a complete event history for all actions performed via the AWS Management Console, SDKs, CLI, and AWS services.

---

## Key Features

### 1. **Event Logging**
- Captures details of API activity, such as who performed the action, when, and from where.
- Records **management events** (e.g., creation, deletion, updates of AWS resources) and optionally **data events** (e.g., object-level access in S3 or Lambda function invocations).

### 2. **Event History**
- Provides up to **90 days** of event history by default for viewing in the CloudTrail console.
- Full logs can be sent to an **S3 bucket** for long-term storage.

### 3. **Integration with Other Services**
- Logs are stored in S3
- Works with **CloudWatch Logs** as an output AWS service for real-time monitoring and **CloudWatch Alarms** for alerts based on specific events.
- Integrates with AWS **Organizations** for centralized auditing across multiple accounts.

### 4. **Insights**
- CloudTrail Insights helps detect unusual activity in your AWS environment, such as spikes in API call volumes or unexpected changes in service usage.

### 5. **Custom Trails**
- Allows the creation of custom trails to monitor specific events or regions.
- Trails can be configured for single or multi-region tracking.

---

## Use Cases

### 1. **Compliance and Auditing**
- Track and log all user and resource activity for compliance frameworks (e.g., GDPR, HIPAA).
- Demonstrates accountability for actions performed in your environment.

### 2. **Security Monitoring**
- Detect unauthorized access or unusual activity.
- Investigate incidents with detailed event logs.

### 3. **Operational Troubleshooting**
- Debug failed API calls or unauthorized actions by reviewing detailed logs.

---

## Exam Tips for AWS SAA-C03

1. **CloudTrail Logs**:
   - Know that CloudTrail logs management events by default but requires enabling data events manually.

2. **Default Retention**:
   - CloudTrail event history is available for **90 days** by default in the console.

3. **Multi-Region Trails**:
   - Understand that custom trails can be set up for all regions to ensure centralized tracking.

4. **Integration with S3 and CloudWatch**:
   - Logs can be stored in S3 for long-term retention and analyzed in real-time using CloudWatch.

5. **CloudTrail Insights**:
   - Useful for identifying anomalies in API usage.

---

## Pricing
- **Management Events**:
  - First copy is free for all regions.
- **Data Events**:
  - Charged based on the number of events recorded.
- **Insights Events**:
  - Additional cost per event.

---

# AWS Config

AWS Config is a service for assessing, auditing, and monitoring the configuration of AWS resources. It helps ensure compliance with policies by tracking resource configurations and evaluating them against rules.

## Key Features

- **Configuration Tracking**: Records and stores resource configurations and changes over time.
- **Compliance Management**: Continuously monitors resources against managed or custom compliance rules.
- **Resource Relationships**: Maps relationships between resources (e.g., EC2 instances and security groups).
- **Custom Rules**: Define custom compliance rules using AWS Lambda.
- **Automated Remediation**: Automatically fixes non-compliant resources via AWS Systems Manager.
- **Integration**: Works with services like CloudTrail and SNS for comprehensive insights and notifications.
- **Multi-Account Support**: Centralized configuration management across accounts with AWS Organizations.

## How It Works

1. **Resource Inventory**: Tracks AWS resources and their configurations.
2. **Rule Evaluation**: Compares configurations against predefined or custom rules.
3. **Notifications**: Sends alerts for non-compliance via Amazon SNS.
4. **Configuration History**: Maintains a history of resource states for auditing and troubleshooting.

## Benefits

- **Visibility**: Detailed insights into resource states and relationships.
- **Compliance Automation**: Continuous evaluation against policies like HIPAA or PCI DSS.
- **Security**: Quickly detect misconfigurations or risks.
- **Audit Support**: Retains configuration history for audits.
- **Operational Efficiency**: Automates issue detection and resolution.

## Use Cases

- **Compliance Monitoring**: Ensure S3 buckets are not public or IAM users have MFA enabled.
- **Change Management**: Detect unauthorized changes to resource configurations.
- **Security Enforcement**: Validate encryption and access controls.

## Managed Rules Examples

- Ensure **S3 buckets have encryption enabled**.
- Verify **IAM users have MFA configured**.
- Confirm **EC2 instances are in a specific VPC**.

## Pricing

- Charges based on:
  - **Configuration Items**: Tracked changes to resources.
  - **Rule Evaluations**: Frequency of compliance checks.
  - **Custom Rules**: Lambda invocation costs.


## Comparison: AWS Config vs CloudTrail vs CloudWatch

| **Feature**              | **AWS Config**                                  | **AWS CloudTrail**                              | **AWS CloudWatch**                               |
|--------------------------|------------------------------------------------|------------------------------------------------|-------------------------------------------------|
| **Purpose**              | Monitors resource configurations.              | Logs API actions and user activity.            | Monitors performance metrics and logs.          |
| **Use Case**             | Compliance and configuration checks.           | Security audits and activity tracking.         | Operational monitoring and alerting.            |
| **Focus**                | Resource state and relationships.              | API activity and user actions.                 | Metrics, logs, and alarms.                      |
| **Data Type**            | Tracks resource configurations.                | Logs API calls and events.                     | Tracks performance metrics, application logs.   |
| **Retention**            | Configuration history in S3.                   | Logs stored in S3 or CloudWatch Logs.          | Metrics and logs based on user-defined policies.|
| **Real-Time Monitoring** | Compliance evaluations in near real-time.       | No real-time monitoring or evaluation.         | Real-time monitoring with alerts and dashboards.|


# Amazon GuardDuty
## Overview
Amazon GuardDuty is a continuous security monitoring and threat detection service that analyzes multiple AWS data sources and uses machine learning, anomaly detection, and integrated threat intelligence to identify unexpected and potentially malicious activity in your AWS environment.

## Key Features

### 1. Data Sources (Input)
- **VPC Flow Logs**
  - Network traffic patterns
  - Communication with malicious IP addresses
  - Unusual data transfer patterns

- **CloudTrail Events**
  - AWS account activity
  - API calls
  - Management events

- **DNS Logs**
  - Domain requests
  - DNS query analysis
  - Communication with known malicious domains

- **Kubernetes Audit Logs** (EKS)
  - Control plane monitoring
  - Cluster activity analysis
  - Container security threats

### 2. Threat Detection
#### Types of Threats Detected
- **Reconnaissance**
  - Port scanning
  - Failed login attempts
  - Unusual API calls

- **Instance Compromises**
  - Cryptocurrency mining (Has a dedicated finding for it)
  - Malware infections
  - Backdoor installations

- **Account Compromises**
  - Unauthorized IAM users
  - Suspicious credential usage
  - Unusual locations

- **Data Exfiltration**
  - Unusual data transfers
  - S3 bucket tampering
  - Suspicious API patterns

### 3. Key Integrations
- **EventBridge**
  - Automated responses
  - Custom workflows
  - Alert routing

- **Security Hub**
  - Centralized findings
  - Compliance status
  - Security scores

- **Detective**
  - Deep investigation
  - Root cause analysis
  - Historical analysis

## Comparison with Other AWS Security Services

| Service | Primary Use Case | Key Differentiator |
|---------|-----------------|-------------------|
| GuardDuty | Threat Detection | ML-powered, continuous monitoring |
| Security Hub | Security Posture | Centralized security management |
| Detective | Investigation | Deep dive analysis and visualization |
| Macie | Data Security | Sensitive data discovery and protection |

# Amazon Inspector
## Overview
- Automated Security Assessments
- For EC2 instances:
    - Leveraging the AWS System Manager (SSM) agent
    - Analyze against unintended network accessibility
    - Analyze the running OS against known vlnerabilities
- For Container Images push to Amazon ECR
    - Asessment of Container Images as they are pushed

- For Lambda Functions
    - Identifies software vulnerabilities in function code and package dependencies
    - Assessment of functions as they are deployed
- Reporting and integration with AWS Security Hub
- Send findings to Amazon Event Bridge

## Evaluated things
- Only for EC2, ECR and Lambda functions
- Continous scanning of the infrastructure, only when needed
- Package vulnerabilities
- Network reachability
- A risk score is associated with all vulnerabilities for prioritization.

# Amazon Macie
## Overview
- Amazon Macie is a fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect your sensitive data in AWS.
- Macie helps identify and alert you to sensitive data, such as PII.

## Use cases
- Analyze S3 to find PII and send to Amazon Event Bridge.
