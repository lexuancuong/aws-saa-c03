# ECR (Elastic Container Registry)
## Overview
- Fully managed Docker container registry service by AWS
- Store, manage, and deploy Docker container images
- Integrated with ECS and other AWS services
- Supports private and public repositories
- Highly available and scalable

## Features
### Storage
- Images stored in S3 behind the scenes
- Pay for storage used by container images
- Supports multiple versions of each image
- Lifecycle policies to manage image versions

### Security
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

### Integration
- Works seamlessly with ECS and EKS
- Supports Docker CLI
- Cross-account access possible through IAM roles
- Supports cross-region replication

## Image Lifecycle
### Pushing Images
```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com

# Tag image
docker tag image:tag aws_account_id.dkr.ecr.region.amazonaws.com/repository:tag

# Push image
docker push aws_account_id.dkr.ecr.region.amazonaws.com/repository:tag
```

### Pulling Images
```bash
# Pull image
docker pull aws_account_id.dkr.ecr.region.amazonaws.com/repository:tag
```

## Lifecycle Policies
- Automatically clean up unused images
- Based on rules you define
- Examples:
  - Remove untagged images
  - Keep only N most recent images
  - Remove images older than X days

## Best Practices
- Use image tagging strategy
  - Avoid using "latest" tag in production
  - Use semantic versioning
- Implement lifecycle policies
- Enable image scanning
- Use immutable tags
- Regularly clean up unused images
- Use cross-region replication for disaster recovery

## Pricing
- Pay for:
  - Storage of images
  - Data transfer OUT
  - API requests
- No cost for:
  - Data transfer IN
  - Data transfer between ECR and ECS in same region

## Use Cases
- Store container images for ECS/EKS deployments
- Share container images across AWS accounts
- Maintain private Docker registry
- Build CI/CD pipelines with container images


# AWS CloudFormation

AWS CloudFormation is an Infrastructure as Code (IaC) service that enables you to define and provision AWS resources using templates.
It helps automate resource creation, configuration, and management in a consistent and repeatable way.

## Key Features
- **Templates**: Define resources in YAML or JSON files.
- **Declarative Syntax**: Specify the desired state of resources without needing procedural code.
- **Resource Stacks**: Deploy and manage a group of resources as a single unit (stack).
- **Change Sets**: Preview the impact of changes before applying updates.
- **Drift Detection**: Identify resource configurations that differ from the template.
- **Cross-Region and Cross-Account**: Manage resources across multiple regions and accounts.

## Benefits
- **Automation**: Simplifies the provisioning and management of resources.
- **Consistency**: Ensures identical infrastructure across environments.
- **Rollback Support**: Automatically rolls back changes if resource creation fails.
- **Cost Management**: Deletes all resources in a stack with a single command.
- **Integration**: Works with AWS services like CodePipeline for CI/CD.

## Use Cases
1. **Environment Provisioning**: Automate the creation of production, staging, or development environments.
2. **Multi-Region Deployment**: Deploy infrastructure in multiple regions with the same template.
3. **Disaster Recovery**: Quickly replicate infrastructure for failover scenarios.
4. **Compliance**: Ensure resources adhere to defined standards.

# Amazon Machine Images (AMI)

## Overview
- Pre-configured templates for EC2 instances
- Contains the operating system, applications, and configurations
- Can be AWS-provided, marketplace, or custom-built
- Regional resource (specific to an AWS region)
- Can be copied across regions

## Types of AMIs
1. **Amazon Quick Start AMIs**
   - Base OS images provided by AWS
   - Regular security patches
   - Free to use (only pay for infrastructure)

2. **AWS Marketplace AMIs**
   - Pre-configured with third-party software
   - Often includes licensing costs
   - Verified by AWS

3. **Community AMIs**
   - Shared by AWS community members
   - Use with caution - no AWS verification
   - Free to use

4. **Custom AMIs**
   - Created from your EC2 instances
   - Include your specific configurations
   - Can be shared across accounts

## AMI Creation Process
```bash
# Basic AMI creation
aws ec2 create-image \
    --instance-id i-1234567890abcdef0 \
    --name "My-Custom-AMI" \
    --description "AMI for production servers"
```

### Steps:
1. Prepare the instance (updates, cleanup)
2. Create AMI (snapshot is taken)
3. Wait for AMI to be available
4. Launch new instances from AMI

## Key Features

### 1. Storage Configuration
- **Root Volume**:
  - EBS or Instance Store backed
  - Contains OS and boot data
  
- **Additional Volumes**:
  - Can include multiple EBS volumes
  - Volume configurations saved in AMI

### 2. Permissions
```json
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": {"AWS": "arn:aws:iam::123456789012:root"},
        "Action": "ec2:LaunchInstance",
        "Resource": "arn:aws:ec2:region::image/ami-id"
    }]
}
```

### 3. Cross-Region Copying
- Copy AMIs to other regions
- Useful for:
  - Disaster recovery
  - Multi-region deployment
  - Geographic expansion

## Best Practices

### Security
1. **Regular Updates**
   - Keep base AMIs current
   - Apply security patches
   - Document changes

2. **Access Control**
   - Restrict AMI sharing
   - Use encryption for sensitive data
   - Regular audits of shared AMIs

### Cost Optimization
1. **Storage Management**
   - Delete unused AMIs
   - Clean up snapshots
   - Use lifecycle policies

2. **Region Strategy**
   - Copy AMIs only when needed
   - Consider transfer costs
   - Clean up copied AMIs

## Common Use Cases
1. **Golden Image Strategy**
   - Standard configurations
   - Compliance requirements
   - Quick deployment

2. **Backup and DR**
   - System recovery
   - Cross-region failover
   - Version control

3. **Auto Scaling**
   - Launch templates
   - Consistent configurations
   - Quick scaling

4. **Migration**
   - Moving workloads
   - Environment replication
   - Cloud migration

## Limitations
- Regional resource
- Size limitations
- Cannot modify after creation
- Cannot delete if in use
- Storage costs for AMIs and snapshots

# Beanstalk
- AWS Elastic Beanstalk is a fully managed service designed to help developers quickly deploy, manage, and scale web applications and services. It simplifies the deployment process by handling infrastructure provisioning, load balancing, auto-scaling, and monitoring while allowing you to focus on writing code.
- With Elastic Beanstalk, you can upload your application code (e.g., Java, .NET, Node.js, Python, Ruby, PHP, or Docker), and the service automatically deploys it to a pre-configured environment. You retain full control over the underlying resources, making it a great choice for developers who want flexibility without the complexity of managing infrastructure.

# Deployment Strategies
## comparison
| **Deployment Strategy**  | **Description**                                                                                   | **Use Case**                                                                            |
|---------------------------|---------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|
| **Recreate Deployment**   | Stops the current version completely before deploying the new version.                           | Simple applications with minimal downtime tolerance.                                   |
| **Rolling Deployment**    | Gradually replaces instances of the old version with the new version in a controlled sequence.   | Applications needing zero downtime with steady resource utilization.                  |
| **A/B Testing Deployment**| Deploys two versions simultaneously to different user groups for testing and comparison.         | Testing user behavior and performance for specific features or interfaces.            |
| **Shadow Deployment**     | Sends live traffic to the new version for testing without affecting end users.                   | Validating new versions under real-world traffic conditions without user impact.       |
| **Immutable Deployment**  | Deploys the new version in entirely new infrastructure while keeping the old version intact.     | Applications requiring high reliability and instant rollback capabilities.             |
| **Rolling with Batches**  | Deploys updates to a subset of instances (batches) at a time, progressively replacing all.        | Balancing speed of deployment with stability, particularly for large-scale systems.    |
| **Elastic Deployment**    | Automatically scales the new version up and the old version down during deployment.              | Applications in environments with auto-scaling and dynamic resource requirements.      |
