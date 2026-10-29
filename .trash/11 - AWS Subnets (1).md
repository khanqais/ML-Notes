---
tags: [aws, subnet]
---
# AWS Subnets

- One subnet = one AZ.
- Must be inside the VPC CIDR, no overlaps.
- **AWS reserves 5 IPs per subnet**: first four and the last one (network, VPC router, DNS, future use, broadcast).
- **Public subnet**: route table has `0.0.0.0/0 -> Internet Gateway`.
- **Private subnet**: no IGW route; outbound via NAT gateway if needed.
- **Auto-assign public IPv4** is a subnet setting.

Related: [[05 - Subnetting]], [[12 - Route Tables and Internet Gateway]]
