---
tags: [aws, advanced]
---
# Advanced VPC

- **VPC Peering**: connect two VPCs (non-transitive, CIDRs must not overlap).
- **Transit Gateway**: hub for many VPCs and on-prem.
- **VPC Endpoints**: Gateway (S3, DynamoDB) and Interface (PrivateLink) to avoid internet/NAT.
- **VPN / Direct Connect**: connect on-prem.
- **Flow Logs**: record accepted/rejected traffic for debugging.
- **Bastion / SSM Session Manager**: access private instances.
- **IPv6** and Egress-only IGW.
- **DNS in VPC**: enableDnsSupport / enableDnsHostnames.
