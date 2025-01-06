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
- Can use CloudWatch and EC2 Reboot CloudWatch Alarm Action to monitor and rebooth the EC2

# Lambda
## Overview
- Serverless compute service, virtual function - no server to manage!
- Run code without provisioning servers
- Pay only for compute time used
- Supports multiple programming languages
- Automatic scaling. When we need to increase ram, it will also increase CPU and network.
- Integrated with many AWS services

## Features
### Limit Execution Per Region
- Memory: 128MB to 10GB
- MAXIMUM EXECUTION TIME: 15 minutes (Important in the exam)
- Environment variables support: 4 KB
- Temporary disk capacity (/tmp): 512MB to 10GB
- Concurrent executions: 1000 (can be increased)

### Integrations
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

### Deployment
- Function packages must be up to 50MB zipped, 250MB unzipped
- Can use Layers to manage dependencies
- Container images up to 10GB

### Combine with other tool Event Bridges or SNS to consume event from RDS

## Security
- Execution Role (IAM Role): Need to add needed permissions to Execution Role so that Lambda can connect to other services. i.e SQS
- Resource-based Policies
- Encryption at rest using KMS
- CloudWatch Logs integration
- Network security with VPC configuration

## Pricing
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

## CloudFront Functions
- Modern applications execute some form of the logic at the edge.
- Edge Function:
    - A code that you write and attach to CloudFront distributions
    - Runs close to your users to minimize latency
- CloudFront provides two types: CloudFront Functions and Lambda@Edge
- Use cases: 
    - Customize the CDN content
    - A/B Testing
## CloudFront Functions
- Lightweight functions written in Javascript
- For high-scale, latency-sensitive CDN customizations
- Sub-ms startup times, milions of requests/second
- Used to change Viewer requests and responses:
    - Viewer Request: after CloudFront receives a request from a viewer
    - Viewer Response: before CloudFront forwards the response to the viewer
- Native feature of Cloudfront

## Lambda@Edge
- Writen in NodeJs or Python
- Scales to 1000s of requests/second
- Used to change CloudFront requests and responses
    - Viewer Request
    - Viewer Response 
    - Origin Request
    - Origin Response
- Author your functions in one AWS Region then CloudFront replicates to its locations

## Comparison between CloudFront Functions and Lambda@Edge
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

## Lambda in VPC
- By default, Lambda function is launched outside your own VPC (in an AWS-owned VPC)
- Therefore, it cannot access resources in your VPC (RDS, ElastiCache, internal ELB,...)
### How to configure Lambda to be in VPC
- You must define the VPC ID, the subnets and the security groups
- Lambda will create an ENI (Elastic Network Interface) in your subnets
- If Lambda functions directly access your database, they may open too many connections under high load => Must use RDS Proxy 
    - Improve scalability by pooling and sharing DB connections
    - Improve availability by reducing by 66% the failover time and preserving connections
    - Improve security by enforcing IAM authentication and storing credentials in Secrets Manager
- The Lambda function must be deployed in your VPC, because RDS Proxy is never publicly accessible

## Step Functions
### Overview
- Serverless visual workflow service
- Orchestrate Lambda functions and other AWS services
- JSON-based Amazon States Language
- Provides visual interface to serverless applications
- Automatically handles errors, retries, and state
- Maximum execution time of 1 year
- Possibility of implementing human approval feature
- Use cases: order fulfillment, data processing, web applications, any workflow

## Use Cases
- Real-time File Processing
- Real-time Stream Processing
- ETL
- Cron Jobs
- Web Applications
- Mobile Backends

## Best Practices
- Keep functions focused and small
- Minimize cold starts
- Use environment variables
- Implement proper error handling
- Monitor and set alarms
- Use versions and aliases
- Optimize memory settings

# ECS (Elastic Container Service)
## Overview
- Amazon ECS is a fully managed container orchestration service that helps you run, stop and manage Docker containers on a cluster
- ECS is a key service for running containerized applications on AWS at scale
- ECS removes the need to install, operate, and scale your own cluster management infrastructure

## Launch Types
- EC2 Launch Type
  - You manage the EC2 instances (infrastructure)
  - Each EC2 instance must run the ECS Agent to register in the ECS Cluster
  - AWS takes care of starting/stopping containers
  - More control over infrastructure, need to manage capacity
  
- Fargate Launch Type
  - Serverless - no EC2 instances to manage
  - You just create task definitions
  - AWS runs containers for you based on CPU/RAM you need
  - Amazon ECS with Fargate launch type is charged based on vCPU and memory resources that the containerized application requests
  - Simpler to use, less control, more expensive

## Key Components
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
  - Cluster – packs instances close together inside an Availability Zone (AZ). 
  - This strategy enables workloads to achieve the low-latency network performance necessary for tightly-coupled node-to-node communication that is typical of HPC applications.


## Integration with other AWS Services
- Application Load Balancer (ALB) for load balancing
- ECR (Elastic Container Registry) for storing Docker images
- CloudWatch for monitoring and logging
- IAM for security and permissions
- VPC for networking

## IAM Roles for ECS
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

## Launch Types Comparison

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

## ECS Service Auto Scaling
### Overview
- Automatically increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses AWS Application Auto Scaling
- ECS Service Auto Scaling (task level) ≠ EC2 Auto Scaling (instance level)
- Scaling based on:
  - ECS Service Average CPU Utilization
  - ECS Service Average Memory Utilization
  - ALB Request Count Per Target

### Auto Scaling Types
- Target Tracking
  - Scale based on target value for a specific CloudWatch metric
  - Example: Keep average CPU utilization at 50%
  
- Step Scaling
  - Scale based on CloudWatch alarms
  - More granular control over scaling behavior
  
- Scheduled Scaling
  - Scale based on specified dates/times
  - Useful for predictable workload changes

### EC2 Launch Type - Additional Scaling
- Auto Scaling Group Scaling
  - Scale EC2 instances based on CPU utilization
  - Add EC2 instances when running out of capacity
  
- Cluster Capacity Provider
  - Automatically provision and scale infrastructure for ECS tasks
  - Capacity Provider paired with an Auto Scaling Group
  - Add EC2 instances when you're missing capacity

### Fargate Auto Scaling
- Simpler scaling as AWS manages infrastructure
- Based on task utilization metrics
- No need to manage EC2 instances
- Scaling policies:
  - Target tracking
  - Step scaling
  - Scheduled scaling

### Service CPU/Memory Metrics
- CloudWatch metrics updated every minute
- Service CPU/Memory utilization
- Service CPU/Memory reservation
- Can be used for target tracking and step scaling

## EventBridge Schedule to Invoke ECS Tasks
### Overview
- EventBridge (CloudWatch Events) can be used to schedule ECS tasks
- Allows running tasks on a schedule without maintaining a continuously running service
- Useful for batch processing, maintenance tasks, or periodic workloads

### Configuration
- Schedule Types
  - Fixed Rate: Run at specified intervals (every N minutes/hours/days)
  - Cron Expression: More complex scheduling patterns
  - One-time scheduled tasks

### Components Required
- ECS Task Definition
  - Defines the container(s) to run
  - Must be registered and active
- ECS Cluster
  - Where the task will run
  - Can be EC2 or Fargate launch type
- IAM Roles
  - EventBridge needs permission to invoke ECS tasks
  - Task needs appropriate permissions for its workload

### Use Cases
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

### Best Practices
- Use Fargate for scheduled tasks to avoid managing infrastructure
- Set appropriate task timeouts
- Monitor task execution and completion
- Use task retries for reliability
- Implement proper error handling and notifications

# EKS (Elastic Kubernetes Service)
## Overview
- Managed Kubernetes service to run Kubernetes on AWS
- Kubernetes is an open-source system for automating containerized applications
- Alternative to ECS, similar goal but different API
- EKS runs upstream Kubernetes, can use all existing Kubernetes tools
- AWS manages the Kubernetes control plane
- Supports both EC2 and Fargate launch types

## Components
### Control Plane
- Managed by AWS
- Runs across multiple AZs for high availability
- AWS handles scaling and updates
- Automated version upgrades and patching
- Key Components:
  - API Server: Entry point for all REST commands
  - etcd: Distributed database storing cluster state
  - Controller Manager: Manages node/replication controllers
  - Scheduler: Assigns pods to nodes based on resource requirements

### Worker Nodes
- EC2 instances that run your containers
- Can use managed node groups
- Can use self-managed nodes
- Can use Fargate (serverless)

### Node Types
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

### How Components Work Together
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

## Data Volumes
- Container Storage Interface (CSI) drivers
- Supports AWS EBS
- Supports Amazon EFS (works with Fargate)
- Supports Amazon FSx for Lustre
- Supports Amazon FSx for NetApp ONTAP

## Networking
- Uses Amazon VPC networking
- Every pod gets an IP address
- Supports AWS VPC CNI (Container Network Interface)
- Can use alternative CNI plugins

## Security
### Authentication
- Uses IAM authentication
- Supports RBAC (Role-Based Access Control)
- Integration with AWS IAM Authenticator

### Network Security
- Security Groups for worker nodes
- Network Policies for pod-to-pod communication
- Integration with AWS Shield and WAF

### Pod Security
- Pod Security Policies
- Security Context
- Container Image Scanning

## Load Balancing
- Supports AWS Application Load Balancer (ALB)
- Supports Network Load Balancer (NLB)
- Automatic integration through Kubernetes service objects

## Monitoring and Logging
- Amazon CloudWatch Container Insights
- AWS CloudTrail for API logging
- Kubernetes native monitoring tools (Prometheus, etc.)
- Support for AWS X-Ray for tracing

## Use Cases
- Microservices Architecture
- Hybrid Deployments
- Batch Processing
- Machine Learning
- Multi-cloud strategies (due to Kubernetes compatibility)

## Comparison with ECS

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

## Best Practices
- Use managed node groups when possible
- Implement proper security controls
- Use cluster autoscaling
- Regular updates and patches
- Proper monitoring and logging
- Use EKS-optimized AMIs
- Implement proper backup strategies


# App Runner
## Overview
- Fully managed service that makes it easy to deploy web applications and APIs
- No infrastructure experience required
- Automatically builds and deploys the web application
- Automatic scaling, highly available
- VPC access support
- Connect to databases, caches, and message queue services

## Source Code Support
### Source Types
- Container Registry
  - Amazon ECR Public
  - Amazon ECR Private
  - Support for Dockerfiles
- Source Code Repository
  - GitHub
  - Support for Python, Node.js, Java, .NET Core

## Features
### Auto Scaling
- Automatic scaling based on requests
- Scale to zero when no requests (cost optimization)
- Configure min/max instances
- Concurrent request-based scaling

### Networking
- Public endpoint by default
- VPC access (connect to RDS, ElastiCache, etc.)
- VPC security groups
- Custom domains with HTTPS

### Security
- IAM integration
- Secrets/Parameter management
- Resource-based policies
- Encryption in transit

### Monitoring
- Amazon CloudWatch integration
- Application logs
- Deployment and service metrics
- Health checks

## Deployment
### Configuration Options
- Memory and CPU configurations
- Environment variables
- Instance size selection
- Auto scaling configuration
- Health check settings

### Deployment Options
- Rolling deployments
- Automatic rollback on failure
- Traffic splitting for testing
- Deployment triggers

## Use Cases
- Web Applications
- APIs and Microservices
- Development and Test Environments
- Small to Medium Applications
- Proof of Concept Projects

## Benefits
- Quick setup and deployment
- No infrastructure management
- Automatic scaling
- Cost-effective (pay for what you use)
- Built-in monitoring and logging
- Integrated security

## Pricing
- Pay for compute and memory resources
- Additional charges for provisioned concurrency
- Data transfer costs apply
- No minimum fees
- Pay only for what you use

## Limitations
- Region availability
- Container size limits
- Concurrent request limits
- Custom runtime limitations
- Network connectivity restrictions

# App2Container
## Overview
- A command-line tool to help you migrate existing applications running on-premises or on EC2 to run in containers
- Supports Java and .NET applications
- Automatically containerizes and migrates applications to Amazon ECS or EKS

## Features
- Analyzes applications and their dependencies
- Generates container images and deployment artifacts
- Creates ECS/EKS deployment configurations
- Integrates with existing CI/CD pipelines

## Process
1. Install App2Container on the source server
2. Analyze applications
3. Extract and containerize applications
4. Deploy to AWS container services (ECS/EKS)

## Benefits
- Simplifies containerization of existing applications
- Reduces manual effort in container migration
- Maintains application dependencies and configurations
- Generates container best practices
