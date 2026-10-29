---
tags: [networking, nat]
---
# NAT (Network Address Translation)

Translates private IPs to public IPs so private hosts can use the internet.

- **Static NAT**: 1 private <-> 1 public (what the AWS Internet Gateway does for instances with public IPs).
- **PAT/NAPT**: many private IPs share one public IP using ports (what a NAT gateway does).
- Outbound allowed, inbound connections from internet blocked.

Related: [[13 - NAT Gateway]]
