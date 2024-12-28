# Cognito
## Overview
- Managed authentication service with multiple authentication methods.
- Add user sign-up and sign-in to applications
- Supports social and enterprise identity providers
- Serverless, scalable, and secure

## Components
### User Pools
- User directory service
- Sign-up and sign-in functionality
- Customizable UI
- Security features:
  - MFA
  - Password policies
  - Email/Phone verification
- Integration with social identity providers
- SAML support for enterprise IdPs

- Can integrate with API Gateway to evaluate Coginito token. Let's say the flow could be:
1. User call retrieve token from Cognito User Pools
2. User send the request with Cognito token to API Gateway
3. API Gateway verify the token with Cognito User Pools
4. Send to Lambda function if the token is valid

### Identity Pools
- Provide temporary AWS credentials
- Enable access to AWS services
- Support anonymous guest access
- Can be used with User Pools
- Fine-grained access control with IAM roles

## Features
### Authentication
- Username/password
- Social identity providers
  - Facebook
  - Google
  - Apple
  - Amazon
- Enterprise identity providers
  - SAML
  - OpenID Connect
- Custom authentication flows

### Security
- Built-in security features
- Encryption in transit and at rest
- Compliance certifications
- AWS WAF integration
- Advanced security features:
  - Adaptive authentication
  - Compromised credential protection
  - Account takeover protection

## Use Cases
- Mobile/Web Applications
- Single Sign-On
- Secure API Access
- Social Identity Federation
- Enterprise Identity Federation

## Best Practices
- Use appropriate authentication flows
- Implement MFA where possible
- Configure password policies
- Use custom domains
- Monitor user activity
- Regular security reviews
- Proper error handling

## Integration
- Works with other AWS services:
  - API Gateway
  - Lambda
  - S3
  - AppSync
  - ALB
- SDKs available for multiple platforms
- REST API support

## Pricing
- Pay per MAU (Monthly Active User)
- Additional charges for:
  - Advanced security features
  - SMS/Email messages
  - SAML federation

# NOTES
## Encryptions
### Encryption in flight (TLS/SSL)
- Data is encrypted before sending and decrypted after receiving
- TLS certificates help with encryptions (HTTPS)
- Encryption in flight ensures no MIMT can happen (man in the middle attack)

### Server-side Encryption 
- Data is encrypted after being received by the server
- Data is decrypted before being sent
- It is stored in an encrypted  form thanks to a key (usually a data key)
- The encryption/decryption keys must be managed somewhare, and the server must have access to it.

### Client-side Encryption
- Data is encrypted by the client and never decrypted by the server 
- Data will be decrypted by a receiving client
- The server should not be able to decrypt the data
- Could levarage Envelope Encryption

# AWS KMS (Key Management Service)

## Overview
AWS Key Management Service (KMS) is a fully managed service that allows you to create, manage, and control cryptographic keys used to secure your data. It integrates with many AWS services and enables secure encryption and decryption operations.

## Key Types
- KMS Keys is the new name of KMS Customer Master Key.
- Symmetric (AES-256 keys)
    - Single encryption key that is used to Encrypt and Decrypt.
    - AWS services that are integrated with KMS use Symmetric CMKs.
    - You never get access to the KMS key unencrypted (must call KMS API to use).
- Asymmetric (RS and ECC key pairts)
    - Public (Encrypt) and Private Key (Decrypt) pair.
    - Used for Encrypt/Decrypt or Sing/Verify operations.
    - The public key is downloadable but you can't access the private key unencrypted.
    - Use case: encryption outside of AWS by users who can't call the KMS API.
## Automatic Key rotation
- AWS-managed KMS Key: automatic every 1 year
- Customer-managed KMS Key: (must be enabled) automatic and on-demand
- Imported KMS Key: only manual rotation possible using alias.

## Types of KMS Keys
- AWS Owned Key (free): SSE-S3, SSE-SQS, SSE-DDB
- AWS Managed Key (free): aws/service-name
- Customer managed keys created in KMS: 1$ / month
- + pay for API call to KMS ($0.03 / 100000 calls)


## Key Policies
Control access to KMS keys, similar to S3 bucket policies.
You cannot control access without them.
1. **Default KMS Policy**
- Created if you don't provide a specific KMS Key Policy
- Complete access to the key to the root user = entire AWS account
   ```json
   {
       "Effect": "Allow",
       "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
       "Action": "kms:*",
       "Resource": "*"
   }
   ```

2. **Key Administrators**
   - Manage keys but can't use them
   - Key deletion, rotation, policies

3. **Key Users**
   - Can use keys for cryptographic operations
   - Encrypt/decrypt data

## Common Use Cases
1. **Envelope Encryption**
   ```
   CMK → Data Key → Actual Data
   ```

2. **S3 Encryption**
   - SSE-KMS for server-side encryption
   - Client-side encryption with KMS

3. **Database Encryption**
   - RDS encryption
   - DynamoDB encryption

4. **Secret Management**
   - Secrets Manager integration
   - Systems Manager Parameter Store

5. Copying Snapshots accross regions
- Copy EBS Volume Encrypted With KMS accross regions
- Attach a KMS Key Policy to authorize cross-account access.

## KMS Multi-Region Keys

### Overview
- Identical KMS keys in different AWS regions that can be used interchangeably.
- Multi-Region keys have the same key ID, key material, automatic rotation,...
- Encrypt in one Region and decrypt in other Regions.
- No need to re-encrypt or making cross-Region API calls.
- KMS Multi-Region are NOT global.
- Each Multi-Region key is managed independently.

## Use cases
### Global client-side encryption
### Encryption on Global DynamoDB
- We can encrypt specific attributes client-side in our DynamoDB table using the **Amazon DynamoDB Encryption Client** (This client call to KMS with low-latency APIs to encrypt and decrypt the data client-side)
### Global Aurora
- We can encrypt specific attributes client-side in our Aurora table using the AWS Encryption SDK
- Combined with Aurora Global Tables, the client-side encrypted data is replicated to other regions
- If we use a multi-region key, replicated in the same region as the Global Aurora DB, then clients in these regions can use lo-latency API calls to KMS in their region to decrypt the data client-side.
- Using client-side encryption we can protect specific fields and guarantee only decryption if the client has access to an API key, we can protect specific fields even from database admins.

### Encrypt and Decrypt S3 with KMS

### AMI Sharing Process Encrypted via KMS
1. AMI in Source Account is encrypted with KMS Key from Source Account.
2. Must modify the image attribute to add a Launch Permission which corresponds to the specified target AWS account.
3. Must share the KMS Keys used to encrypted the snapshot the AMI references with the target account/IAM Role
4. The IAM Role/User in the target account must have the permissions to DescribeKey, ReEncrypted, CreateGrant, Decrypt.
5. When lauching an EC2 instance from the AMI, optionally the target account can specify a new KMS key in its own account to re-encrypt the volumes.

## Limitations
- 4KB data size limit for direct encryption
- API request quotas
- Key deletion minimum waiting period (7-30 days)
- Regional service

# SSM Parameter Store
## Overview
Centralized storage and management of your secrets and configuration data such as passwords, database strings and license codes. You can encrypt values or store as plain text and secure access at every level.

## Features
- Secure storage for configuration and secrets
- Serverless, scalable, durable, easy SDK
- Optional Seamless Encryption using KMS
- Version tracking of configurations/secrets
- Security through IAM
- Notification with Amazon EventBridge
- Integration with CloudFormation
- SSM Parameter Store Hierachy (Organization with as much as directories as possible.)

### Parameter Store Tiers Comparison

| Feature | Standard | Advanced |
|---------|----------|-----------|
| Total parameters allowed (per AWS account and Region) | 10,000 | 100,000 |
| Maximum size of parameter value | 4 KB | 8 KB |
| Parameter policies available | No | Yes |
| Cost | Free | $0.05 per advanced parameter per month |
| Storage Pricing | Free | $0.05 per advanced parameter per month |
| API Throughput | 40 TPS | 100 TPS (higher throughput available on request) |
| Integration with AWS Services | Yes | Yes |
| Secure String Parameter Support | Yes | Yes |
| Hierarchical Storage | Yes | Yes |
| Change Notifications | Yes | Yes |
| Version History | Latest version only | Up to 100 versions |
| Parameter Policies Available | No | Yes - Expiration, Notification, NoChangeNotification |
| Automatic AWS KMS Encryption | Optional | Optional |
| Cross-Region Support | No | Yes |
| Tags Support | Yes | Yes |

### Parameter Policies (Advanced Tier Only)
Can assign multiple policies at a time
1. **Expiration**
   - Delete parameters after specific time
   - Useful for temporary credentials

2. **ExpirationNotification**
   - EventBridge notification before expiration
   - Set notification threshold (e.g., 15 days)

3. **NoChangeNotification**
   - Alert if parameter hasn't changed
   - Detect stale configurations
4. **Assign a TTL to a param**
    - Allow to assign to TTL to a parameter (expiration date) to force updating or deleteing sensitive data such as password.

## Comparison with Secret Manager
| **Criterion**                   | **Use SSM Parameter Store**                    | **Use AWS Secrets Manager**                     |
|----------------------------------|-----------------------------------------------|-----------------------------------------------|
| **Cost**                         | Cheaper for basic storage of secrets.          | More expensive due to added features.          |
| **Automatic Rotation**           | Not natively supported.                       | Supported for AWS services like RDS.           |
| **Integration**                  | Integrates well with basic AWS services.       | Deeper integration for secrets with managed rotation. |
| **Secret Management Complexity** | Suitable for simple use cases.                | Best for complex secret management needs.      |


# AWS Secrets Manager

## Overview
AWS Secrets Manager is a secrets management service that helps you protect access to your applications, services, and IT resources. It enables you to rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle.

## Key Features

### 1. Secret Rotation
- **Automatic Rotation**
  - Built-in rotation for AWS services
  - Custom rotation using Lambda functions
  - Configurable rotation schedules
```python
# Example rotation configuration
{
    "RotationRules": {
        "AutomaticallyAfterDays": 30,
        "Duration": "2h",
        "ScheduleExpression": "cron(0 16 1,15 * ? *)"
    }
}
```

### 2. Fine-Grained Access Control
- **IAM Policies**
  - Resource-based policies
  - Identity-based policies
  - VPC endpoint policies
```json
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": [
            "secretsmanager:GetSecretValue"
        ],
        "Resource": "arn:aws:secretsmanager:region:account-id:secret:secret-name"
    }]
}
```

### 3. Encryption
- **At Rest**
  - AWS KMS encryption
  - Customer managed keys support
- **In Transit**
  - TLS 1.2 encryption
  - HTTPS endpoints

## Multi-Region Secrets
- Replicate Secrets across multiple AWS Regions
## Common Use Cases

### 1. Database Credential Management
```plaintext
Flow:
1. Application requests secret
2. Secrets Manager returns current credentials
3. Application connects to database
4. Automatic rotation occurs in background
```

### 2. Application Secrets
- API keys
- OAuth tokens
- Encryption keys
- Configuration secrets

### 3. Cross-Account Access
- Multi-region applications
- Share secrets across accounts
- Centralized secret management
- Disaster recovery strategies
- Ability to promote a read replica Secret to a standalone Secret

## Best Practices

### 1. Security
- Enable automatic rotation
- Use minimum required permissions
- Monitor secret access
- Implement secret versioning

### 2. Cost Optimization
- Delete unused secrets
- Optimize API calls
- Use caching when appropriate

### 3. Operational Excellence
- Tag secrets appropriately
- Use meaningful names
- Document rotation procedures
- Set up monitoring and alerts

## Pricing
- Per secret stored
- Per 10,000 API calls
- No additional charge for:
  - Automatic rotation
  - Encryption
  - Replication

## Comparison with Secret Manager
| **Criterion**                   | **Use SSM Parameter Store**                    | **Use AWS Secrets Manager**                     |
|----------------------------------|-----------------------------------------------|-----------------------------------------------|
| **Cost**                         | Cheaper for basic storage of secrets          | More expensive due to added features          |
| **Automatic Rotation**           | Not natively supported                        | Native rotation for AWS services & custom Lambda |
| **Integration**                  | Basic AWS service integration                 | Deep integration with RDS, Redshift, DocumentDB |
| **Secret Management Complexity** | Simple parameter/secret storage               | Complex secret rotation & lifecycle management |
| **Price**                        | Free for standard tier                        | $0.40 per secret per month + $0.05 per 10k API calls |
| **Use Case**                     | Configuration and simple secrets              | Database credentials & complex secrets         |
| **Maximum Size**                 | 4KB (Standard) / 8KB (Advanced)              | 64KB                                          |
| **Encryption**                   | Optional KMS encryption                       | Mandatory KMS encryption                      |
| **Cross-Region Replication**     | No native support                            | Supported                                     |
| **Resource Policy**              | No                                           | Yes                                           |#

# AWS Certificate Manager (ACM)
## Overview
- Easily provision, manage, and deploy TLS Certificates
- Provide in-flight encryption for websites (HTTPS)
- Support both public and private TLS certificates
- Free of charge for public TLS certificates
- Automatic TLS certificate renewal
- Integrations with (load TLS certificate on)
    - ELB
    - CDN
    - APIs on API Gateway
- CANNOT use ACM with EC2 (can't be extracted)

## Feature
- Option to generate the certificate outside of ACM and then import it
- No automatic renewal, must import a new certificate before expiry
- ACM sends daily expiration events starting 45 days prior to expiration
- AWS Config has a managed rule named acm-certificate-expiration-check to check for expiring certificates

## ACM - Integration with API Gateway
- Create a Custom Domain Name in API Gateway
- Edge-optimized (default): For global clients
    - Requests are routed through the CloudFront Edge locations (imporoves latency)
    - The API Gateway still lives in only one region
    - THE TLS CERTIFICATE MUST BE IN THE SAME REGION AS CLOUDFRONT.
    - Then setup CNAME or (better) A-Alias record in Route53
    => As the Edge-Optimized API Gateway is using a custom AWS managed CloudFront distribution behind the scene to route requests across the globe through CloudFront Edge locations, the ACM certificate must be created in `us-east-1`.

- Regional:
    - For clients within the same region
    - The TLS Cert must be imported on API Gateway in the same region as the API Stage.
    - Then setup CNAME or A-Alias record in Route53.

# AWS WAF (Web Application Firewall)

## Overview
AWS WAF is a web application firewall that helps protect web applications from common web exploits (Layer 7) that could affect application availability, compromise security, or consume excessive resources.

## Integrations
- ALB
- API Gateway
- CloudFront
- AppSync GraphQL API
- Cognito User Pool

## Features
Configure Web ACL for these Rule Actions:
  - IP Set: up to 10,000 IP addresses - use multiple Rules for more IPS
  - SQL injection and CSS XSS
  - size constraints, geo-match (block countries)
  - Rate-based rules - for DDoS protection
  - Regex Pattern Sets (Match custom pattern)
- Web ACL are Regional except for CloudFront
- CIDR notation
- PCI DSS requirements
- HIPAA security rules


## Managed Rules
### AWS Managed Rules
- Core rule set (CRS)
- Admin protection
- Known bad inputs
- SQL database rules
- Linux operating system rules
- PHP application rules
- WordPress application rules

### Marketplace Rules
- Third-party vendor rules
- Specialized protection
- Industry-specific rules

## Rule Groups
- Reusable collections of rules
- Can be shared across web ACLs
- Managed or custom rule groups
```json
{
    "Name": "CustomRuleGroup",
    "Capacity": 100,
    "Rules": [
        {
            "Name": "BlockBadUserAgents",
            "Priority": 1,
            "Statement": {
                "ByteMatchStatement": {
                    "SearchString": "BadBot",
                    "FieldToMatch": {
                        "SingleHeader": {
                            "Name": "user-agent"
                        }
                    },
                    "TextTransformations": [
                        {
                            "Priority": 1,
                            "Type": "LOWERCASE"
                        }
                    ],
                    "PositionalConstraint": "CONTAINS"
                }
            },
            "Action": {
                "Block": {}
            }
        }
    ]
}
```

## Pricing
- Web ACL capacity units (WCU)
- Rule evaluations
- CAPTCHA charges
- Bot control charges
- Logs delivery

## Common Problems
### Fixed IP while using WAF with a Network Load Balancer (layer 4)
- WAF doesn't support the Netwrok Load Balancer
- We can use Global Accelerator for fixed IP and WAF on the ALB

## Limitations
- Rule capacity limits
- Rule evaluation order
- Regional service
- Integration restrictions

# AWS Shield 
## Overview
- This service protects from DDoS with network and infrastructure-level protection (layer 3 and 4)
## Types
### AWS Shield Standard:
- Free service that is activated for every AWS customer
- Provides protection from attacks such as SYN/UDP Floods, Reflection attack and other layer 3 and layer 4 attacks
### AWS Shield Advanced
- Optional DDoD mitigation server (3000$ per month per Organization)
- 24/7 access to AWS DDoD response team (DRP)
- Protect against higher fees during usage spike due to DDoS
- Shield Advanced automatic application layer DDoS mitigation automatically creates, evaluates and deploy AWS WAF rules to mitigate layer 7 attacks.

# AWS Firewal Manager
## Overview
- Manage rules in all accounts of an AWS Organization in one place (this service).
- Security policy: common set of security rules:
    - WAF rules (Application Load Balancer, API Gateways, CloudFront)
    - AWS Shield Advanced (ALB, CLB, NLB, Elastic IP, CloudFront)
    - Security Groups for EC2, ALB and ENI resources in VPC
    - AWS Network Firewal (VPC Level)
    - Route 53 Resolver DNS Firewall
    - Policies are created at the region level
- Rules are applied to new resources as they are created (good for compliance) across all future accounts in your Organization.

## Integrations
Combine with AWS WAF and AWS Shield to maximizing the protection and management.
