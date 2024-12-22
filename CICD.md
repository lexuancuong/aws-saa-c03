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
