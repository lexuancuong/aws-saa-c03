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
