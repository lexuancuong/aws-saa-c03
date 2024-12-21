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


