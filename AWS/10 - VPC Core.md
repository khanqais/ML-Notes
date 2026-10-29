---
tags: [aws, vpc]
---
# VPC (Virtual Private Cloud)

Your own logically isolated private network inside AWS.

## You control
- IP range (CIDR, /16 to /28)
- Subnets
- Route tables
- Gateways
- Security (SG, NACL)

## Components
| Component | Purpose |
|---|---|
| CIDR block | Address space |
| Subnet | Slice in one AZ |
| Route table | Traffic rules |
| Internet Gateway | Internet access |
| NAT Gateway | Outbound-only for private |
| Security Group | Instance firewall |
| NACL | Subnet firewall |
| Elastic IP | Static public IPv4 |
| VPC Endpoint | Private access to AWS services |

Related: [[11 - AWS Subnets]], [[15 - Default VPC]]
