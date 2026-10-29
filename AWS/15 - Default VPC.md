---
tags: [aws, vpc]
---
# Default VPC

Each region comes with one:
- CIDR `172.31.0.0/16`
- One default subnet per AZ, each `/20`
- Internet Gateway attached, route to IGW
- Default security group
- Instances get public IPs by default

Good for quick tests; for production create a custom VPC.

Related: [[10 - VPC Core]]
