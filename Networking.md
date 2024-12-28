# API Gateway

## Overview
- Fully managed service for creating, publishing, and managing APIs
- Acts as a "front door" for applications to access data, business logic, or functionality
- Handles all the tasks involved in accepting and processing up to hundreds of thousands of concurrent API calls
- Serverless and scalable
- Supports RESTful APIs and WebSocket APIs

## Types of APIs
- HTTP API (Newer, lower latency, lower cost)
  - Built for REST APIs
  - Proxy to Lambda, HTTP backends
  - No data transformations
  - Simpler features, cheaper pricing
- REST API (More features)
  - Full API management features
  - API keys, request/response transformations
  - OIDC and OAuth2
  - Request validation
- WebSocket API
  - Real-time two-way communication
  - Maintains persistent connections
  - Common for chat apps, streaming dashboards

## Endpoint Types
### Edge-Optimized (default)
- Best for global clients
- Requests are routed through CloudFront Edge locations
- API Gateway still lives in one region
- Improves latency for global clients
- Great for public-facing APIs
- Supports only REST APIs

### Regional
- For clients in the same region
- Could manually combine with CloudFront
- Use when:
  - Need more control over caching strategies and distribution
  - Clients are in the same region
  - Want to manage your own CloudFront distribution
- Supports both HTTP and REST APIs

### Private
- Can only be accessed from your VPC using VPC endpoint (ENI)
- Use VPC endpoint policies to define access
- Access through:
  - VPC endpoints (must create VPC endpoint)
  - Resource policies to define access
- Use when:
  - APIs should only be accessible from within your VPC
  - Need complete isolation of your APIs
- Supports only REST APIs

### Endpoint Selection Considerations
- Edge-Optimized: Global users, simple setup needed
- Regional: 
  - Same-region clients
  - Custom CloudFront setup needed
  - HTTP API requirement
- Private: 
  - Internal services only
  - Maximum security needed
  - VPC isolation required

## Key Features
- Security
  - IAM roles and policies
  - Cognito user pools
  - Lambda authorizer
  - API keys and usage plans
  - AWS WAF integration
- Monitoring & Logging
  - CloudWatch integration
  - Access logs
  - Execution logs
  - Metrics & Alarms
- Development Features
  - API versioning
  - Multiple environments (dev, prod)
  - Swagger/OpenAPI support
  - Request/Response transformations
  - CORS support
  - Authentication

## Integration Types
- Lambda Function
  - Invoke Lambda functions
  - Most popular integration
- HTTP
  - Integration with HTTP endpoints
  - Internal HTTP APIs
  - Public HTTP APIs
- AWS Services
  - Direct integration with AWS services
  - Example: SQS, SNS, Step Functions
## Authentication
### User Authentication through
- IAM Roles (useful for internal applications)
- Cognito (identity for external users - example mobile users)
- Custom Authorizer (your own logic)

## Caching
- Reduces number of calls to backend
- Default TTL: 300 seconds (min: 0s, max: 3600s)
- Caching capacity: 0.5GB to 237GB
- Cache is defined per stage
- Can encrypt cache
- Cache key customization available

## Throttling
- Account limit: 10,000 requests per second
- Can set Stage limit and Method limits
- Burst limit: 5,000 requests
- Excess requests get 429 error (Too Many Requests)

## Pricing
- HTTP API
  - $1.00 per million requests
  - No minimum fees
- REST API
  - $3.50 per million requests
  - Data transfer charges apply
- WebSocket API
  - $1.00 per million messages
  - $0.25 per million connection minutes

## Common Use Cases
- Serverless API backends
- Microservices architecture
- Mobile app backends
- B2B integrations
- Internal API services

## Best Practices
- Use appropriate API type for use case
- Implement caching when possible
- Use custom domains
- Enable CloudWatch logging
- Set up monitoring and alerts
- Use API keys for client identification
- Implement proper security controls
- Use models for request/response validation
