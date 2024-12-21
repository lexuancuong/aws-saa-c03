# EC2

## Configuration
- OS
- CPU - RAM
- Storage space
  - EBS
  - EFS
- Network card
- Firewall rules
- Bootstrap script
  - Configure at first launch
    - EC2 user data
  - Run with sudo permission
- Type
  - t2.micro
    - 1 CPU 1 Gi RAM
  - t2.xlarge (base)
    - 4 CPU 16Gi RAM
  - 4x.large
    - 16 CPU 32Gi RAM

## Security Group
### Overview
- Control how traffic is allowed into or out of our EC2 instances
- Security groups only contain `allow` rules
- Security groups rules can reference by IP by other security groups
- One security group can be attached to multiple EC2 instances

### Configuration
- Common ports
  - 21 -> FTP
  - 22 -> SSH
  - 22 -> SFTP
  - 80 -> HTTP
  - 443 -> HTTPS
- IPs
  - 0.0.0.0 -> means anywhere

### Tips
- It's good to maintain one separate security group for SSH access
- If your application is not accessible(timeout), this is a security group issue
- Connection refused -> application issue
- All inbound traffic are blocked by default
- All outbound traffic is authorized by default
- If 2 EC2 instances have the same security group, they can connect to each others -> Common in load balancer

## EC2 Instances Purchasing Options

### On-demand Instances
#### Cost
- Has the highest cost but no upfront payment
- Linux&Windows: Pay for minutes Others: pay by hour

#### Recommended
- Short workload, predictable pricing pay by second
- Short-term and uninterrupted workloads

### Reserved Instances
#### Cost
- Up to 72% discount compared to on-demand instances
- Convertible Reserved (can change instance type, family OS, scope and tenancy) Instance discount up to 66%

#### Specification
- Could reserve 1 year or 3 years with specific instances in Region or AZ
- Reserve a specific instance (type, region, tenancy and OS)
- No upfront, Partial Upfront and All upfront
- Can buy and sell in the reserved instance marketplace

#### Recommended
- Steady state usage application

### Saving plans
#### Cost
- Usage beyond EC2 saving plans is billed at on-demand price
- Get a discount based on long-term usage (up to 70%)

#### Specification
- Locked to specific instance family (i.e m5d) and AWS region
- Commit to a certain type of usage 10$ for 1 or 3 years
- Beyond EC2 Savings plans is billed at the On-Demand price
- Flexible across instance size, OS and tenancy

### EC2 Spot Instances
#### Cost
- Discount of up to 90% compared to On-Demand
- The most cost-efficient instances in AWS

#### Specification
- Instance that you can lose at any point of time if your max price is less than the current spot price

#### Recommended
- Useful for resilient workloads
- SHOULDN'T use for critical workloads

### EC2 Dedicated Hosts
#### Cost
- The most expensive option

#### Specification
- Address compliance requirements and use existing server-bound software licenses
- Reserved for 1 or 3 years

#### Recommended
- Useful for software that have complicated licensing model

### EC2 Dedicated Instances
#### Cost
- Expensive like EC2 dedicated hosts

#### Specification
- Instances run on hardware that's dedicated to you
- May share hardware with other instances in same account
- No control over instance placement (can move hardware after Stop/Start)

### EC2 Capacity Reservations
#### Cost
- You're charged at On-Demand rate whether you run instances or not

#### Specification
- Reserve On-Demand instances capacity in a specific AZ for any duration
- Always have access to EC2 capacity when you need it
- No time commitment (create/cancel anytime) no billing discounts
- Combine with Regional Reserved Instances and Saving Plans to benefit from billing discounts

#### Recommended
- Suitable for short-term, uninterrupted workloads that need to be in a specific AZ

## Tips
### Balance between kind of usage
- Computation
- Memory
- Networking

## Lambda
### Overview
- Serverless compute service, virtual function - no server to manage!
- Run code without provisioning servers
- Pay only for compute time used
- Supports multiple programming languages
- Automatic scaling. When we need to increase ram, it will also increase CPU and network.
- Integrated with many AWS services

### Features
#### Limit Execution Per Region
- Memory: 128MB to 10GB
- Maximum execution time: 15 minutes
- Environment variables support: 4 KB
- Temporary disk capacity (/tmp): 512MB to 10GB
- Concurrent executions: 1000 (can be increased)

##### Integrations
- API Gateway
- Kinesis
- DynamoDB
- S3
- CloudFront
- CloudWatch Events/EventBridge
- CloudWatch Logs
- SNS
- SQS
- Cognito

##### Deployment
- Function packages must be up to 50MB zipped, 250MB unzipped
- Can use Layers to manage dependencies
- Container images up to 10GB

##### Combine with other tool Event Bridges or SNS to consume event from RDS

#### Security
- Execution Role (IAM Role)
- Resource-based Policies
- Encryption at rest using KMS
- CloudWatch Logs integration
- Network security with VPC configuration

#### Pricing
- Pay per request
  - First 1 million requests per month are free
  - $0.20 per 1 million requests thereafter
- Pay per compute time (in 1ms increments)
  - Free tier: 400,000 GB-seconds of compute time per month
  - $0.0000166667 per GB-second
  - Example: using 512MB = 0.5GB
    - Running for 1 second = $0.0000166667 * 0.5 = $0.00000833
    - Running for 30 seconds = $0.00000833 * 30 = $0.00025
- Additional charges:
  - If using Lambda in VPC, duration of ENI lifecycle
  - Network traffic when accessing internet or other AWS services
  - CloudWatch Logs storage for Lambda logs

#### Use Cases
- Real-time File Processing
- Real-time Stream Processing
- ETL
- Cron Jobs
- Web Applications
- Mobile Backends

#### Best Practices
- Keep functions focused and small
- Minimize cold starts
- Use environment variables
- Implement proper error handling
- Monitor and set alarms
- Use versions and aliases
- Optimize memory settings

#### CloudFront Functions
- Modern applications execute some form of the logic at the edge.
- Edge Function:
    - A code that you write and attach to CloudFront distributions
    - Runs close to your users to minimize latency
- CloudFront provides two types: CloudFront Functions and Lambda@Edge
- Use cases: 
    - Customize the CDN content
    - A/B Testing
##### CloudFront Functions
- Lightweight functions written in Javascript
- For high-scale, latency-sensitive CDN customizations
- Sub-ms startup times, milions of requests/second
- Used to change Viewer requests and responses:
    - Viewer Request: after CloudFront receives a request from a viewer
    - Viewer Response: before CloudFront forwards the response to the viewer
- Native feature of Cloudfront

##### Lambda@Edge
- Writen in NodeJs or Python
- Scales to 1000s of requests/second
- Used to change CloudFront requests and responses
    - Viewer Request
    - Viewer Response 
    - Origin Request
    - Origin Response
- Author your functions in one AWS Region then CloudFront replicates to its locations

##### Comparison between CloudFront Functions and Lambda@Edge
| Feature | CloudFront Functions | Lambda@Edge |
|---------|---------------------|-------------|
| Runtime | JavaScript | Node.js and Python |
| Number of Requests | Millions of requests per second | Thousands of requests per second |
| Maximum Execution Time | < 1ms | 5-10 seconds |
| Maximum Memory | 2MB | 128MB to 10GB |
| Total Package Size | 10KB | 50MB (1MB for viewer events) |
| Network Access | No | Yes |
| File System Access | No | Yes |
| Access to Request Body | No | Yes |
| Pricing | Free tier available, then $0.10 per 1M invocations | No free tier, charged per request and duration |
| Use Cases | Cache key normalization, URL rewrites/redirects, Request header manipulation and authorization | Access to external services, Complex processing, A/B testing, complex code (depends on a 3rd libraries)|
| Supported Events | Viewer Request, Viewer Response | Viewer Request, Viewer Response, Origin Request, Origin Response |
| Cold Starts | No cold starts (sub-millisecond) | Yes (>100ms) |

#### Lambda in VPC
- By default, Lambda function is launched outside your own VPC (in an AWS-owned VPC)
- Therefore, it cannot access resources in your VPC (RDS, ElastiCache, internal ELB,...)
##### How to configure Lambda to be in VPC
- You must define the VPC ID, the subnets and the security groups
- Lambda will create an ENI (Elastic Network Interface) in your subnets
- If Lambda functions directly access your database, they may open too many connections under high load => Must use RDS Proxy 
    - Improve scalability by pooling and sharing DB connections
    - Improve availability by reducing by 66% the failover time and preserving connections
    - Improve security by enforcing IAM authentication and storing credentials in Secrets Manager
- The Lambda function must be deployed in your VPC, because RDS Proxy is never publicly accessible




