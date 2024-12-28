# IAM (Identity and Access Management)
## Core Components
### Users
- Represents people or services needing AWS access
- Has permanent long-term credentials
- Can belong to multiple groups
- Best practice: one IAM user per person/service.

### Groups
- Collection of IAM users
- Users inherit permissions from group
- Cannot nest groups (no groups within groups)
- Simplifies permission management
- Best practice: assign permissions via groups (an then assign users to group), not individual users.

### Roles
- Temporary identity with permissions
- Used by:
  - AWS services (EC2, Lambda)
  - External users (Federation)
  - Cross-account access
- No permanent credentials
- Automatically rotated
- Best practice: use roles for applications on EC2

### Policies
To simplify: IAM roles provide the identity (who or what can perform actions), while IAM policies define the permissions (what actions can be performed on resources).
#### Types
1. **Identity-based Policies**
   - Attached to users, groups, or roles
   - Define allowed/denied permissions

2. **Resource-based Policies**
   - Attached to resources (S3 buckets, SQS queues)
   - Define who can access the resource
   - Contains Principal element

3. **Permission Boundaries**
   - Sets maximum permissions
   - Used to delegate administration
   - Doesn't grant permissions by itself

#### Policy Evaluation Logic
- Explicit Deny > Explicit Allow > Default Deny
- Cross-account: both accounts must allow the action
- Resource-based policy can allow cross-account access without role assumption
- Check on the whole polices: IAM's policies (Resource-based policy, Identity-based policy, Permission boundaries) and AWS Organization's police (SCP (Service Control Policy) and session policy).
    - If having explicit allow without deny => Allow
    - If having explicit allows with a deny => Deny

## Advanced IAM Policies

### Policy Variables
- Use dynamic values in policies
- Common variables:
  - ${aws:username}
  - ${aws:userid}
  - ${aws:PrincipalTag/key-name}
  - ${aws:CurrentTime}
  - ${aws:SecureTransport}
```json
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": "s3:PutObject",
        "Resource": "arn:aws:s3:::mybucket/${aws:username}/*"
    }]
}
```

### Condition Elements
1. **String Conditions**:
   - StringEquals
   - StringNotEquals
   - StringLike (with wildcards)
   ```json
   "Condition": {
       "StringEquals": {
           "aws:RequestedRegion": "us-east-1"
       }
   }
   ```

2. **Numeric Conditions**:
   - NumericEquals
   - NumericGreaterThan
   - NumericLessThanEquals
   ```json
   "Condition": {
       "NumericLessThanEquals": {
           "s3:max-keys": "10"
       }
   }
   ```

3. **Date Conditions**:
   - DateEquals
   - DateGreaterThan
   - DateLessThan
   ```json
   "Condition": {
       "DateGreaterThan": {
           "aws:CurrentTime": "2023-12-31T23:59:59Z"
       }
   }
   ```

### Resource-Based vs Identity-Based
1. **Identity-Based**:
   - Attached to IAM users/roles/groups
   - Control what the identity can do
   ```json
   {
       "Effect": "Allow",
       "Action": ["s3:GetObject"],
       "Resource": "arn:aws:s3:::mybucket/*"
   }
   ```

2. **Resource-Based**:
   - Attached to resources
   - Control who can access the resource
   - Include Principal element
   ```json
   {
       "Effect": "Allow",
       "Principal": {"AWS": "arn:aws:iam::ACCOUNT-ID:user/username"},
       "Action": ["s3:GetObject"],
       "Resource": "arn:aws:s3:::mybucket/*"
   }
   ```
   There are some services don't support Resource-Based. We need to set up IAM role (Identify-based policy) instead. i.e: Kinesis Stream, EC2 Auto Scaling, ECS task...

### Permission Boundaries
- IAM Permission Boundaries are supported for users and roles (not groups).
- Advanced feature to use a managed policy to set the maximum permissions an IAM entity can get.
- Used for delegation
- Example: Limit developers to specific services
```json
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": [
            "s3:*",
            "cloudwatch:*"
        ],
        "Resource": "*"
    }]
}
```

### Session Policies
- Used with temporary credentials
- Further restricts role permissions
- Session policies cannot grant permissions. They only constrain the maximum permissions for the session.
```json
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Action": ["s3:GetObject"],
        "Resource": "arn:aws:s3:::mybucket/readonly/*"
    }]
}
```

### Common Policy Scenarios
1. **Multi-Region Access Control**:
```json
{
    "Condition": {
        "StringEquals": {
            "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
        }
    }
}
```

2. **IP-Based Access**:
```json
{
    "Condition": {
        "IpAddress": {
            "aws:SourceIp": ["192.0.2.0/24", "203.0.113.0/24"]
        }
    }
}
```

3. **Time-Based Access**:
```json
{
    "Condition": {
        "DateGreaterThan": {"aws:CurrentTime": "2023-01-01T00:00:00Z"},
        "DateLessThan": {"aws:CurrentTime": "2024-01-01T00:00:00Z"}
    }
}
```
4. Restrict access to accounts that are members of an AWS Organization
Use aws:PrincipalOrgID

## Security Features
### Authentication Methods
- Access Keys (CLI/API access)
- Console password
- MFA options:
  - Virtual MFA (Google Authenticator)
  - U2F Security Keys (YubiKey)
  - Hardware MFA device
  - SMS text messages

### Best Practices
1. **Root Account**
   - Use only for initial setup
   - Enable MFA
   - Don't create access keys
   - Store credentials securely

2. **Password Policy**
   - Minimum length
   - Require specific character types
   - Password expiration
   - Prevent password reuse

3. **Access Management**
   - Principle of least privilege
   - Use groups for permissions
   - Regular access review
   - Remove unused credentials
   - Enable CloudTrail for auditing

## Important Features for SAA-C03
1. **Cross-Account Access**
   - Using roles
   - Resource-based policies
   - External ID for third parties

2. **Service Control Policies (SCPs)**
   - Used with AWS Organizations
   - Restricts permissions across accounts

3. **Instance Profiles**
   - Container for IAM role
   - Used by EC2 instances
   - Automatically manages temporary credentials

4. **Security Tools**
   - IAM Credentials Report
   - IAM Access Analyzer
   - Service Last Accessed Data

## Common Use Cases
1. **Application Security**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Action": [
         "s3:GetObject",
         "s3:PutObject"
       ],
       "Resource": "arn:aws:s3:::mybucket/*"
     }]
   }
   ```

2. **Cross-Account Access**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [{
       "Effect": "Allow",
       "Principal": {
         "AWS": "arn:aws:iam::ACCOUNT-ID:root"
       },
       "Action": "sts:AssumeRole",
       "Condition": {
         "StringEquals": {
           "sts:ExternalId": "unique-id"
         }
       }
     }]
   }
   ```

## Exam Tips
- Understand policy evaluation logic
- Know when to use roles vs. users
- Remember resource-based policy capabilities
- Understand cross-account access patterns
- Know IAM security best practices
- Understand temporary credentials
- Remember IAM is global service
- Know common troubleshooting scenarios

# AWS Organizations

AWS Organizations is a service for managing and consolidating multiple AWS accounts under a single organization.
It simplifies governance, cost management, and security at scale by enabling centralized control.
This service is a free service.
**IAM Groups** are useful within a single AWS account for managing users and permissions, like organizing users by roles (e.g., admins, developers).
**AWS Organizations** is required when managing multiple AWS accounts. 
You use it to consolidate billing, apply service control policies (SCPs), and automate the management of cross-account permissions.

## Key Features

- **Account Management**: Group and manage AWS accounts within an organization.
- **Service Control Policies (SCPs)**: Apply policies to control access to AWS services across accounts.
- **Consolidated Billing**: Combine billing for multiple accounts into a single payment for cost efficiency.
- **Organizational Units (OUs)**: Organize accounts into hierarchies for targeted management.
- **Integrated Services**: Works with AWS services like Config, CloudTrail, and IAM Identity Center.

## Benefits

- **Centralized Governance**: Manage accounts, permissions, and budgets centrally.
- **Cost Optimization**: Share volume discounts and Reserved Instance (RI) benefits across accounts.
- **Scalability**: Easily add or remove accounts as the organization grows.
- **Security Compliance**: Enforce policies and restrictions on AWS usage across all accounts.
- **Improved Isolation**: Separate workloads and environments (e.g., production vs. development).

## Key Concepts

- **Management Account**: The primary account that creates and manages the organization.
- **Member Accounts**: AWS accounts that belong to the organization and are governed by policies.
- **Service Control Policies (SCPs)**: JSON-based policies applied at the organization or OU level to restrict services or actions.
- **Organizational Units (OUs)**: Logical groupings of accounts to apply specific policies.

## Use Cases

1. **Centralized Billing**:
   - Consolidate payments for multiple accounts to streamline cost management.

2. **Access Control**:
   - Restrict actions like creating EC2 instances in certain regions using SCPs.

3. **Environment Separation**:
   - Use separate accounts for production, staging, and development environments.

4. **Cost Allocation**:
   - Track and allocate costs to specific teams or departments by account.

5. **Compliance Enforcement**:
   - Ensure all accounts adhere to security and compliance requirements.

## Example: SCP to Deny EC2 Use in Specific Regions
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-west-1", "us-west-2"]
        }
      }
    }
  ]
}
```

## Key Differences
1. **Scope of Control**
   - IAM: Within a single AWS account
   - Organizations: Across multiple AWS accounts

2. **Purpose**
   - IAM: Identity and access management within account
   - Organizations: Account management and governance

3. **Policy Application**
   - IAM: Directly affects user/resource permissions
   - Organizations: Sets boundaries for entire accounts

4. **Use Cases**
   - IAM: Day-to-day access control
   - Organizations: Enterprise-wide governance

# IAM Identity Center (formerly AWS SSO)

## Overview
- Centrally manage workforce access to multiple AWS accounts and applications
- Single sign-on service
- FREE service
- Built on top of AWS Organizations
- Regional service

## Key Features
### Single Sign-On
- One set of credentials for all AWS accounts
- Access AWS Management Console
- Access cloud applications
- Access SAML 2.0-enabled applications
- Command Line Interface (CLI) access

### Identity Sources
1. **Built-in Identity Store**
   - Default identity source
   - Manage users and groups directly
   - Free to use

2. **Active Directory**
   - AWS Managed Microsoft AD
   - Self-managed AD (via AD Connector)
   - Connect existing corporate directory

3. **External Identity Provider**
   - Okta
   - Azure AD
   - Any SAML 2.0 compatible IdP

### Permission Sets
- Collections of policies
- Define access levels
- Reusable across accounts

### Fine-grained Permissions and Assignments
1. **Assignment Structure**
   - User/Group → Permission Set → AWS Account
   - One user can have multiple permission sets
   - Same permission set can be assigned to multiple accounts

2. **Permission Levels**
   - Job-function policies (ViewOnlyAccess, PowerUserAccess)
   - Custom permission sets
   - AWS managed policies
   - Customer managed policies

3. **Assignment Strategies**
   - Assign by team (Dev, Ops, Security)
   - Assign by environment (Prod, Dev, Test)
   - Assign by project
   - Time-based access (temporary assignments)

4. **Multi-Account Permissions**
   ```
   Developer Group → ReadOnly Permission Set → Development Account
                  → PowerUser Permission Set → Test Account
                  → ViewOnly Permission Set → Production Account
   ```
## Key Benefits
1. **Security**
   - Centralized access control
   - MFA enforcement
   - Detailed audit logs
   - Fine-grained permissions

2. **User Experience**
   - Single login for all resources
   - No password management per account
   - Intuitive user portal
   - Browser and CLI support

3. **Administration**
   - Centralized user management
   - Automated account provisioning
   - Easy permission management
   - Integration with existing identity providers

## Common Use Cases
1. **Enterprise Access Management**
   - Multiple AWS accounts
   - Different departments/teams
   - Various permission levels

2. **Contractor Access**
   - Temporary access
   - Limited permissions
   - Easy deprovisioning

3. **Application Access**
   - Cloud applications
   - Internal tools
   - Third-party services

# AWS Directory Services

## Overview
Microsoft Active Directory (AD) is a directory service developed by Microsoft that provides a centralized platform for managing and securing access to resources in a networked environment
AWS Directory Service provides multiple ways to use Microsoft Active Directory (AD) with other AWS services. It's crucial for enterprise identity management and SSO.

## Types of Directory Services

### 1. AWS Managed Microsoft AD
- Actual Microsoft AD running on AWS
- Supports Group Policy
- Create your own AD in AWS, manage users locally, supports MFA
- Best for:
  - Organizations needing actual AD features
  - Hybrid cloud scenarios
  - Applications requiring AD

### 2. AD Connector
- Directory gateway (proxy) to on-premises AD
- Users are manged on the on-premise AD
- Best for:
  - Using existing AD with AWS services
  - Keeping all directory data on-premises
  - Single sign-on scenarios

### 3. Simple AD
- AD-compatible managed directory on AWS
- Standalone directory powered by Samba 4
- Basic AD features
- Best for:
  - Simple user/group management
  - Small organizations (<5000 users)
  - Basic AD compatibility

## Common Use Cases
1. **EC2 Domain Join**
   - Join EC2 instances to AD domain
   - Centralized management
   - Group Policy application

2. **SSO to AWS Services**
   - Integration with IAM Identity Center
   - Access AWS Console with AD credentials
   - Unified access management

3. **Workspaces Integration**
   - AD authentication for virtual desktops
   - User profile management
   - Group Policy settings

## Key Features Comparison

| Feature | Managed AD | AD Connector | Simple AD |
|---------|------------|--------------|-----------|
| Scale   | Large      | Large        | Small     |
| Trust Relationships | Yes | No | No |
| Group Policy | Yes | Yes (proxied) | Basic |
| MFA | Yes | Yes | No |
| Price | $$$ | $$ | $ |

## Best Practices
1. Choose the right directory type based on needs
2. Plan network connectivity (VPC, VPN, Direct Connect)
3. Implement high availability across AZs
4. Regular backups and monitoring
5. Use secure protocols (LDAPS)

# AWS Control Tower
## Overview
- Easy way to set up and govern a secure and compliant multi-account AWS environment based on best practices
- Works as preventive or detective to control the security and compliant of our environments.
- AWS Control Tower uses AWS Organizations to create accounts

## Benefits
- Automate the set up of your environment in a few clicks
- Automate ongoing policy management using guardrails
- Detect policy violations and rememdiate them
- Monitor compliance through an interactive dashboard

## Components
### Guardrails
- Provides ongoing governance for your Control Tower environemtn AWS Accounts
    - Preventive Guardrail - using SCPs (e.g Restrict Regions across all your accounts)
    - Detective Guardrail - using AWS Config (e.g identify untagged resources)
