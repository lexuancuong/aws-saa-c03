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

# VPC
## Prequesites
### CIDR - IPV4
- Classless Inter-Domain Routing - a method for allocating IP address
- Used in Security Groups rules and AWS networking in general
- They help to define an IP address range.
- Has 2 components:
    - Base IP (Represents an IP contained in the range)
    - Subnet Mask (How many bits can change in the IP)
#### Subnet Mask Represention Logic
- The Subnet Mask basically allows part of the underlying IP to get additional next values from the base IP (i.e 192.168.0.0)
- Subnet masks are represented by a "/" followed by a number (e.g., /32, /24, /16)
- The number after "/" represents how many bits are fixed in the IP address
- Formula: 2^(32-subnet) = number of IP addresses available

##### Common CIDR Blocks in AWS VPC
| CIDR Block   | # of IPs | Typical Use Case |Comments
|--------------|----------|------------------|-----------------------------
| /28          | 16 (2^4)       | Small subnet     |192.168.0.0 -> 192.168.0.15
| /24          | 256 (2^8)     | Standard subnet  |192.168.0.0 -> 192.168.0.255
| /20          | 4,096    | Large subnet     |192.168.0.0 -> 192.168.16.255
| /16          | 65,536   | Entire VPC       |192.168.0.0 -> 192.168.255.255
| /0           |   All    | All VPC | -

##### Important Notes for AWS SAA-C03
- VPC CIDR block must be between /16 and /28
- CIDR blocks must not overlap between VPCs if you plan to peer them
- Cannot change VPC CIDR block size after creation
- Each subnet must be within VPC CIDR block
- Reserve first 4 and last IP address in each subnet (AWS requirement)

### Private IP
- The Internet Assigned Numbers Authority (IANA) established certain blocks of IPv4 addresses for the use of private (LAN) and public (Internet addresses).
- Private IP can only allow certain values:
    - 10.0.0.0 - 10.255.255.255 (10.0.0.0/8) <- in big networks
    - 172.16.0.0 - 172.31.255.255 <- AWS default VPC in that range
    - 192.168.0.0 - 192.168.255.255 <- home networks
- All the rest of the IP addresses on the Internet are Public.

## Overview
- Virtual Private Cloud
- Support multiple VPCs in an AWS region (max 5/ region - soft limit, can increase)
- Max. CIDR per VPC is 5, for each CIDR:
    - Min.size is /28 (16 IP addresses)
    - Max.size is /16 (65536 addresses)
- Only private IPv4 ranges are allowed (10.0.0.0, 172.16.0.0, 192.168.0.0)
- Your VPC CIDR should NOT overlap with your other networks.
- All new AWS accounts have a default VPC.
- New EC2 instances are lauched into the default VPC if no subnet is specified.
- Default VPC has Internet connectivity and all EC2 instances inside it have public IPv4 addresses.

## Subnet (IPv4)
- There could be public subnets and private subnet within an availability Zone.
- AWS reserves 5 IP addresses (first 4 and last 1) in each subnet.
- These 5 IP addresses are not available for use and can't be assigned to an EC2 instance
- Example: if CIDR block 10.0.0.0/24, then reserved IP addresses are:
    - 10.0.0.0 - network Address
    - 10.0.0.1 - reserved by AWS for the VPC router
    - 10.0.0.2 - reserved by AWS for mapping to Amazon-provided DNS
    - 10.0.0.3 - reserved by AWS for future use
    - 10.0.0.255 - network broadcase address. AWS doesn't support broadcase in a VPC, therefore the address is reserved
- **EXAM TIP**: If you need 29 IP addresses for EC2 instances: You can't choose a subnet of size / 27 because 32 - 5 (reserved IPs)= 27 < 29.

### Key Differences

| Feature | Public Subnet | Private Subnet |
|---------|--------------|----------------|
| Internet Access | Direct access via IGW | Only through NAT Gateway/Instance |
| Route Table | Has route to IGW (0.0.0.0/0 → IGW) | No direct route to IGW |
| Use Cases | - Web servers<br>- Load balancers<br>- Bastion hosts | - Databases<br>- Application servers<br>- Backend systems |
| IP Addressing | Resources can have public IPs | Only private IP addresses |
| External Access | Can be accessed from internet | No direct internet access |

## Internet Gateway (IGW)
- Allows communication between VPC and the Internet 
(For public-facing resources that need two-way internet communication)
- Horizontally scaled, redundant, and highly available by default
- No availability risk or bandwidth constraints
- Must be created separately from a VPC
- One VPC can only be attached to one IGW and vice versa
- Supports IPv4 and IPv6
- Notes: When configurating Route Table, need to forwarding request from root router to the Internet Gateway.

### Key Characteristics
1. **One IGW Per VPC**
   - Only one IGW can be attached to a VPC at a time
   - IGW cannot be detached from a VPC while there are active AWS resources in the VPC (such as EC2 instances or Load Balancers)

2. **State**
   - Can be either 'attached' or 'detached' to a VPC
   - Acts as a gateway only when attached

3. **High Availability**
   - Automatically redundant and highly available
   - No need to create multiple IGWs for fault tolerance

### Requirements for Internet Access
1. **VPC Level**
   - Must have an IGW attached
   - Route table must have a route to IGW

2. **Subnet Level**
   - Subnet must be public (route table includes route to IGW)
   - Route table must have a route to 0.0.0.0/0 (all IPv4) pointing to IGW

3. **Instance Level**
   - Must have a public IP or Elastic IP
   - Security group must allow relevant traffic
   - NACL must allow traffic

### Route Table Configuration Example
```
Destination     Target
0.0.0.0/0       igw-id    # Route to Internet Gateway
10.0.0.0/16     local     # Local route for VPC
```

### Common Use Cases
1. **Public Subnets**
   - Web servers
   - Bastion hosts
   - NAT Gateways

2. **Outbound Internet Access**
   - Software updates
   - External API calls
   - File downloads

3. **Inbound Internet Access**
   - Web applications
   - Public APIs
   - Customer-facing services

### Exam Tips (SAA-C03)
1. **Remember**
   - IGW is required for internet connectivity
   - One IGW per VPC
   - Automatically highly available
   - Both IPv4 and IPv6 support

2. **Common Scenarios**
   - Public subnet configuration
   - Internet access requirements
   - VPC design questions
   - Cost optimization

3. **Troubleshooting**
   - Check IGW attachment
   - Verify route table entries
   - Confirm public IP assignment
   - Review security group rules

### Limitations
- One IGW per VPC
- Cannot attach to multiple VPCs

## Bastion Hosts
A Bastion Host (also known as Jump Box) is a server used to manage access to internal or private networks from an external network (usually the internet).
(For administrators to securely access private resources)

### Key Characteristics
1. **Placement**
   - Located in a public subnet
   - Acts as entry point to private resources
   - Only instance exposed to the internet

2. **Security**
   - Highly secured and hardened
   - Limited access (typically SSH/RDP only)
   - Restricted security group rules
   - Regular security patches and updates

### Architecture
```plaintext
Internet → IGW → Bastion Host (Public Subnet) → Private Resources (Private Subnet)
```

### Common Use Cases
1. **SSH Access to Private EC2**
    Must allow inbound from internet on port 22
   ```plaintext
   User → SSH to Bastion (Port 22) → SSH to Private EC2
   ```

2. **RDP Access to Windows Instances**
   ```plaintext
   User → RDP to Bastion (Port 3389) → RDP to Private Instance
   ```

3. **Database Administration**
   ```plaintext
   DBA → Bastion → Private RDS/Database
   ```

## NAT (Network Address Translation)
Allows instances in private subnets to access the internet while preventing the internet from initiating connections to these instances.
(For private resources that only need outbound internet access)


### NAT Instance (Legacy)
- EC2 instance in a public subnet configured to route traffic
- Must disable EC2 Source/Destination Check
- Must have Elastic IP attached
- Route table: private subnet → NAT Instance

#### Limitations
- Single point of failure
- Manual scaling and management
- Limited bandwidth based on EC2 instance type
- Must manage security groups and rules

### NAT Gateway (Recommended)
- Fully managed AWS service
- Created in a specific AZ
- Requires an IGW
- 5 Gbps bandwidth that can scale up to 45 Gbps
- No security groups needed

#### Key Points for SAA-C03
1. **High Availability**
   - Create NAT Gateway in each AZ (Achive fault tolerance)
   - Route tables per AZ must point to local NAT Gateway

2. **Cost Optimization**
   - Charged per hour of usage and data processing
   - More expensive than NAT Instance but more reliable

3. **Common Architecture**
```plaintext
Private Subnet → NAT Gateway (Public Subnet) → IGW → Internet
```

### NAT Gateway vs NAT Instance
| Feature | NAT Gateway | NAT Instance |
|---------|------------|--------------|
| Availability | Highly available in AZ | Manual setup required (Depends on EC2)|
| Bandwidth | Up to 45 Gbps | Depends on instance type |
| Maintenance | Managed by AWS | Self-managed |
| Cost | Per hour & data processing | EC2 charges |
| Security Groups | Not supported | Required |
| Bastion Server | Cannot be used as bastion | Can be used as bastion |

### Exam Tips
1. **Remember**
   - NAT Gateway is preferred over NAT Instance
   - Need one NAT Gateway per AZ for HA
   - Always deployed in public subnet

2. **Cost Scenarios**
   - Development: NAT Instance (cheaper)
   - Production: NAT Gateway (reliable)

3. **Troubleshooting**
   - Check route tables
   - Verify IGW exists
   - Ensure NAT Gateway is in public subnet
   - Confirm Elastic IP attachment

## IGW, NAT and Bastion Memory
- IGW is like your building's main entrance (two-way traffic)
- NAT Gateway is like a mail room (only outgoing mail)
- Bastion Host is like a security desk (controlled staff entry)

## Network Access Control Lists (NACLs)
Network ACLs are an optional layer of security that acts as a firewall for controlling traffic in and out of subnets.

### Key Characteristics
1. **Subnet Level**
   - One NACL per subnet
   - New subnets are assigned the default NACL
   - Multiple subnets can share one NACL
   - Operates at subnet boundary

2. **Rule Structure**
   - Rules are evaluated in number order (1-32766)
   - Lower number = higher priority
   - First matching rule is applied
   - Last rule is always * (deny all)
   - Must specify both inbound and outbound rules

3. **Stateless**
   - Must define rules for both inbound and outbound traffic
   - Response traffic must be explicitly allowed
   - Different from security groups which are stateful

### Ephemeral Ports and NACLs
- Ephemeral ports (dynamic ports) are temporary ports assigned by the operating system for client-side communication.
- Clients connect to a defined port, and expect a response on an ephemeral port

#### Understanding Ephemeral Ports
1. **Port Ranges**
   - Windows: 49152-65535
   - Linux: 32768-60999
   - AWS recommends: 1024-65535

### Default NACL
- Accepts everything inbound/outbout with the subnets it's associated with
- Do NOT modify the default NACL instead create custom NACLs
```plaintext
Inbound Rules:
Rule # | Type    | Protocol | Port Range | Source    | Allow/Deny
100    | All IPv4| All     | All        | 0.0.0.0/0 | ALLOW
*      | All IPv4| All     | All        | 0.0.0.0/0 | DENY

Outbound Rules:
Rule # | Type    | Protocol | Port Range | Dest      | Allow/Deny
100    | All IPv4| All     | All        | 0.0.0.0/0 | ALLOW
*      | All IPv4| All     | All        | 0.0.0.0/0 | DENY
```

### Common NACL Rules Example
- THE LESS RULE NUMBER AN NETWORK ACL RULE HAS, THE MORE PRIORITY IT HAS.
```plaintext
Inbound Rules:
100 ALLOW HTTP (80) from 0.0.0.0/0
110 ALLOW HTTPS (443) from 0.0.0.0/0
120 ALLOW SSH (22) from trusted IPs
130 ALLOW Return Traffic (1024-65535) from 0.0.0.0/0
* DENY all traffic

Outbound Rules:
100 ALLOW HTTP (80) to 0.0.0.0/0
110 ALLOW HTTPS (443) to 0.0.0.0/0
120 ALLOW Response Traffic (1024-65535) to 0.0.0.0/0
* DENY all traffic (#Note: Deny everything if it isn't explicitly declared)
```

### NACL vs Security Groups
| Feature | NACL | Security Group |
|---------|------|----------------|
| Level | Subnet level | Instance level |
| Rule Type | Allow and Deny rules | Allow rules only |
| State | Stateless (return traffic must be explicitly allowed by rules)| Stateful (return traffic is automatically allowed, regardless of any rules of the outbound port)|
| Rule Processing | Rules processed in order | All rules evaluated |
| Return Traffic | Must be explicitly allowed | Automatically allowed |
| Scope | All instances in subnet | Individual instances |

### Best Practices
1. **Rule Numbering**
   - Leave gaps between rule numbers (100, 200, 300)
   - Easier to insert new rules later
   - Start with 100 as first rule

2. **Security**
   - Use NACLs as subnet-level firewall
   - Block specific IP addresses (blacklisting)
   - Additional defense layer with security groups

3. **Troubleshooting**
   - Check rule numbers and order
   - Verify both inbound and outbound rules
   - Ensure ephemeral ports are allowed
   - Consider connection tracking

## VPC Peering
VPC Peering is a networking connection between two VPCs that enables routing traffic between them privately using IPv4 or IPv6 addresses.

### Key Characteristics
1. **Connection Properties**
   - Non-transitive (A ↔ B, B ↔ C doesn't mean A ↔ C)
   - Not region-restricted (can peer across regions)
   - No overlapping CIDR blocks allowed
   - No edge to edge routing through peering

2. **Limitations**
   - Maximum of 125 peering connections per VPC
   - Cannot create a peering connection between VPCs with matching or overlapping CIDR blocks
   - Cannot edit or modify a peering connection after it's created

### Common Use Cases
1. **Resource Sharing**
   - Share resources across different VPCs
   - Access services in different AWS accounts
   - Connect production and development environments

2. **Multi-Account Strategies**
   - Connect VPCs across different AWS accounts
   - Maintain security boundaries while allowing communication

3. **Cross-Region Applications**
   - Connect applications across regions
   - Implement disaster recovery setups

### Configuration Steps
1. **Peering Setup**
```plaintext
VPC A (Requester) → Create Peering Connection → VPC B (Accepter)
```

2. **Route Tables**
   - UPDATE ROUTE TABLES IN BOTH VPCS
   - Add routes pointing to the peered VPC's CIDR
   - Example:
```plaintext
VPC A Route Table:
Destination: VPC B CIDR → Target: pcx-xxxxxx (Peering Connection)

VPC B Route Table:
Destination: VPC A CIDR → Target: pcx-xxxxxx (Peering Connection)
```

### Security Considerations
1. **Network ACLs and Security Groups**
   - Each VPC maintains its own security controls
   - Security groups can reference peered VPC security groups
   - NACLs remain subnet-specific

2. **DNS Resolution**
   - Can enable DNS resolution across peering connection
   - Allows private DNS resolution between VPCs

## VPC Endpoints
VPC Endpoints allow you to privately connect your VPC to supported AWS services without requiring an Internet Gateway, NAT device, VPN connection, or AWS Direct Connect.

### Main Benefits
- Improved Security: Traffic stays within AWS network
- Lower Latency: Direct connection to services
- No bandwidth constraints
- No data transfer charges when accessing AWS services
- No need for public IP addresses

### Types of Endpoint

1. **Interface Endpoints (Powered by AWS PrivateLink)**
   - Uses ENI (Elastic Network Interface) with private IP
   - Supports most AWS services
   - Charged per hour and per GB of data processed
   - Examples of supported services:
     - API Gateway
     - CloudFormation
     - CloudWatch
     - SQS, SNS, SES
     - KMS
     - ECS
     - ECR

2. **Gateway Endpoints**
   - Uses route table entries
   - ONLY SUPPORTS S3 AND DYNAMODB
   - Free of charge
   - Regional only
   - More highly available

### Key Differences

| Feature | Interface Endpoint | Gateway Endpoint |
|---------|-------------------|------------------|
| How it works | ENI with private IP | Route table entry |
| Supported services | Most AWS services | Only S3 and DynamoDB |
| Cost | Hourly rate + data processing | Free |
| Security | Security groups | VPC endpoint policies |
| Availability | AZ-specific | Highly available |
| Access | Private IP address | Route table entry

### Note
- Gateway is most likely going to be preferred all the time at the exam
- Cost: free for Gateway, $ for interface endpoint

## VPC Flow Logs
### Overview
- Capture information about IP traffic going into your interfaces:
    - VPC Flow Logs
    - Subnet Flow Logs
    - Elastic Network Interface (ENI) Flow Logs
- Helps to monitor and troubleshoot connectivity issues.
- Flow logs data can go to S3, CloudWatch Logs, and Kinesis Data Firehose.
- Captures network information from AWS managed interfaces too: ELB, RDS, ElasticCache, Redshift, Workspaces, NATGW, TransitGateway,...

### Components
- `srcaddr` and `dstaddr`: Help identify problematic IP.
- `srcport` and `dstport`: help identity problematic ports.
- `Action`: success or failure of the request due to Security Group / NACL.
- Can be used for analytics on usage pattern or malicious behavior
- Query VPC flows logs using Athena on S3 or CloudWatch Logs Insights.

### Integrations
- Integrate with Cloudwatch Logs and CloudWatch Contributor Insights to find top IP addresses caused the issue.
- Integrate with CloudWatch Logs and CW Alarm to fire an alarm.
- Integrate with S3 Bucket and query on Amazon Athena and visualize with Amazon Quicksight.

## AWS Site-to-Site VPN
### Overview
- Connects on-premises network to AWS VPC through encrypted VPN connection.
- Traffic goes over the public internet.
- Automatically encrypted using IPSec protocol.
- Two VPN tunnels per connection for redundancy.
- Quick to set up (can be ready in hours).

### Components
1. **Virtual Private Gateway (VGW)**
   - VPN concentrator on the AWS side
   - Attached to the VPC
   - Can be attached to only one VPC at a time
   - Supports IPSec protocol
   - Automatically highly available

2. **Customer Gateway (CGW)**
   - Software application or physical device on customer side
   - On-premises router configuration
   - Can use static or dynamic routing (BGP)
   - Public IP needed (static) for on-premises device

### Key Features
1. **Route Propagation**
   - Enable route table propagation in VGW
   - Routes automatically added to route tables
   - Simplifies network management

2. **Routing Options**
   - Static Routing: Manually specify network ranges
   - Dynamic Routing (BGP): Automatically exchange routes
   - BGP ASN needed for dynamic routing

3. **High Availability**
   - Two tunnels per connection
   - Each tunnel terminates on different AWS endpoints
   - Use multiple Customer Gateways for redundancy

### Important Specifications
1. **Speed & Bandwidth**
   - Maximum throughput: 1.25 Gbps
   - Support for IPv4 and IPv6
   - Can establish multiple VPN connections for more bandwidth

2. **Monitoring**
   - CloudWatch metrics available
   - Tunnel status monitoring
   - Data transfer monitoring
   - Error monitoring

### Common Use Cases
1. **Hybrid Cloud**
   - Connect on-premises data center to AWS
   - Extend corporate network into cloud
   - Backup and disaster recovery

2. **Multi-Site Connectivity**
   - Connect multiple office locations
   - Branch office connectivity
   - Remote site access

3. **Development and Testing**
   - Dev/test environments in AWS
   - Access to on-premises resources
   - Staged migrations

### Security Considerations
1. **Encryption**
   - IPSec encryption by default
   - Data-in-transit protection
   - Key rotation and management

2. **Network Security**
   - Security groups and NACLs still apply
   - Can implement additional firewall rules
   - Monitor traffic patterns

### Cost Components
1. **VPN Connection Hours**
   - Hourly charge for each VPN connection
   - No data transfer in charges
   - Data transfer out charges apply

2. **Additional Costs**
   - Customer Gateway (on-premises hardware)
   - Network bandwidth
   - On-premises infrastructure

### Best Practices
1. **Design**
   - Plan for redundancy
   - Consider bandwidth requirements
   - Document IP address ranges

2. **Implementation**
   - Test failover scenarios
   - Monitor both tunnels
   - Keep configurations in sync

3. **Operations**
   - Regular monitoring
   - Backup configurations
   - Update security patches

### Limitations
- Maximum throughput 1.25 Gbps
- Latency can be variable (uses internet)
- Customer gateway device compatibility
- Need public IP for customer gateway

### When to Use Site-to-Site VPN
1. **Good Choice When**
   - Quick implementation needed
   - Lower cost solution acceptable
   - Can tolerate internet-based connectivity
   - Lower bandwidth requirements

2. **Consider Alternatives When**
   - Need consistent latency
   - Higher bandwidth required
   - Require dedicated private connection
   - In these cases, AWS Direct Connect might be better

### Exam Tips (SAA-C03)
1. **Remember**
   - Two tunnels for redundancy
   - VGW attaches to one VPC only
   - Need public IP for customer gateway
   - Supports both static and dynamic routing

## AWS Direct Connect (DX)
### Overview
- Dedicated private physical connection between on-premises and AWS
- Connection is private, reliable, and high bandwidth
- Takes months to establish the connection
- Data travels over private network (not internet)
- More expensive than Site-to-Site VPN but better performance

### Connection Types
1. **Dedicated Connection**
   - 1Gbps, 10Gbps, and 100Gbps capacity
   - Physical ethernet port dedicated to a customer
   - Request made to AWS first, then completed by AWS Direct Connect Partners

2. **Hosted Connection**
   - 50Mbps, 500Mbps, to 10Gbps
   - Connection requests are made via AWS Direct Connect Partners
   - Capacity can be added or removed on demand
   - 1, 2, 5, 10 Gbps available at select AWS Direct Connect Partners

### Key Components
1. **Direct Connect Location**
   - Physical location where AWS maintains Direct Connect equipment
   - Usually in partner data centers
   - Must have presence at location or work with partner

2. **Virtual Interfaces (VIF)**
    We need to setup a Virtual Private Gateway on your VPC.
   - **Private VIF**: Connect to resources in a VPC
   - **Public VIF**: Connect to AWS public services (S3, DynamoDB)
   - **Transit VIF**: Connect to resources through Transit Gateway

### High Availability Options
1. **Single Connection**
   - One DX connection
   - Can add Site-to-Site VPN as backup

2. **Multiple Connections**
   - Multiple DX connections for redundancy
   - Can be at same or different locations
   - Recommended for production environments

3. **Hybrid Configuration**
   ```plaintext
   Primary: Direct Connect
   Backup: Site-to-Site VPN (for failover)
   ```

### Use Cases
1. **Large Dataset Transfer**
   - Big data analytics
   - Media content transfer
   - Regular database migration

2. **Real-time Applications**
   - Financial trading
   - Voice/video conferencing
   - Real-time data feeds

3. **Hybrid Environments**
   - Extended corporate network
   - Consistent application performance
   - Regular data center access

### IMPORT ADDONS*
#### Direct Connect Gateway
- A Direct Connect Gateway allows you to connect your Direct Connect connection to one or more VPCs in different regions (same account or different accounts).
- Acts as a central hub for connecting multiple VPCs and on-premises networks.
- Enables global connectivity through a single Direct Connect connection.
- Supports both private and transit virtual interfaces.

#### Direct Connect Encryption
- Data in transit is not encrypted but is private
- AWS Direct Connect + VPN provides an IPsec-encrypted private connection
- Good for an extra level of security but slightly more complex to put in place

### Direct Connect - Resiliency
- High Resiliency for Critical Workloads
- Maximum Resilience is achieved by separate connections terminating on separate decives in more than one location. => For short: Setup 2 AWS Direct Connection per corporate data center.

### Site-to-Site VPN connection as a backup
- In case Direct Connect fails, you can set up a backup Direct Connect connection (expensive) or a Site-to-Site VPN connection.

### Key Features
1. **Performance**
   - Consistent network performance
   - Reduced network costs
   - Increased bandwidth throughput
   - Lower latency

2. **Security**
   - Private network connection
   - Does not use public internet
   - Data never exposed to public
   - Supports encryption (optional)

3. **Compatibility**
   - Works with all AWS services
   - Supports both IPv4 and IPv6
   - Compatible with all AWS regions

### Direct Connect vs Site-to-Site VPN

| Feature | Direct Connect | Site-to-Site VPN |
|---------|---------------|------------------|
| Speed | 1Gbps - 100Gbps | Up to 1.25Gbps |
| Latency | Consistent | Variable |
| Path | Private network | Public internet |
| Setup time | Months | Hours |

# AWS Transit Gateway (TGW)

## Overview
• For having transitive peering between thousands of VPC and on-premises, hub-and-spoke (star) connection
• Regional resource, can work cross-region
• Share cross-account using Resource Access Manager (RAM)
• You can peer Transit Gateways across regions
• Route Tables: limit which VPC can talk with other VPC
• Works with Direct Connect Gateway, VPN connections
• Supports IP Multicast (not supported by any other AWS service)

## Key Features

### 1. Network Connectivity
- Connect thousands of VPCs and on-premises networks
- Hub-and-spoke (star) architecture
- Transitive peering between networks
- Route tables to control traffic flow
- Support IP multicast

### 2. Types of Attachments
- VPC attachments
- Direct Connect Gateway attachments
- VPN attachments (Site-to-Site)
- Peering attachments (Cross-region)
- Connect to other transit gateways

### 3. Routing
- Route tables at the transit gateway level
- Route propagation can be enabled/disabled
- Support static and dynamic routing
- Blackhole routes for security

## Common Architectures

### 1. Centralized Router
```plaintext
VPC1 → Transit Gateway → VPC2
         ↑
         ↓
      VPC3, VPC4, On-premises
```

### 2. Isolated VPCs
```plaintext
Dev VPCs → Transit Gateway (Route Table 1)
Prod VPCs → Transit Gateway (Route Table 2)
```

### 3. Shared Services
```plaintext
Application VPCs → Transit Gateway → Shared Services VPC
                        ↓
                  On-premises
```

## Use Cases

### 1. VPC Consolidation
- Simplify complex VPC peering
- Reduce connection count
- Centralized management

### 2. Multi-Account AWS Architecture
- Connect VPCs across accounts
- Share resources efficiently
- Maintain security boundaries

### 3. Global Network Architecture
- Connect multiple regions
- Centralize internet egress
- Distributed application deployment

### 4. Hybrid Cloud Networking
- Connect on-premises with AWS
- Extend corporate network
- Consistent connectivity

## Limitations
- Regional resource (but can peer across regions)
- Maximum bandwidth per VPC connection: 50 Gbps
- 5000 attachments per transit gateway
- Maximum of 10,000 routes per route table

## Pricing
- Hourly charge for each attachment
- Data processing charges per GB
- Additional charges for:
  - VPN attachments
  - Peering connections
  - Inter-region traffic

## Integration with Other Services
### 1. Direct Connect
- Can attach Direct Connect gateway
- Provides private connectivity
- Supports multiple Direct Connect connections

### 2. Site-to-Site VPN
- Supports multiple VPN connections
- Can use with Direct Connect as backup
- Automatic failover capability

### 3. VPC
- Attach multiple VPCs
- Control routing between VPCs
- Share resources across VPCs

### 4. Route 53 Resolver
- DNS resolution across networks
- Centralized DNS management
- Hybrid DNS solution

## Transit Gateway vs Other Solutions

| Feature | Transit Gateway | VPC Peering | Direct Connect |
|---------|----------------|--------------|----------------|
| Scalability | Thousands of connections | Limited peer-to-peer | Limited by physical connections |
| Architecture | Hub-and-spoke | Mesh | Point-to-point |
| Transitive Routing | Yes | No | No |
| Cross-Region | Yes (peering) | Yes | Yes (with gateway) |
| Bandwidth | Up to 50 Gbps per connection | Up to 100 Gbps | 1-100 Gbps |
| Setup Complexity | Medium | Low | High |
| Cost | Pay per attachment and data transfer | Only data transfer | Higher fixed costs |
