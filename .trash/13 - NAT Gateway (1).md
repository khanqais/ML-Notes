---
tags: [aws, nat]
---
# NAT Gateway

- Lives in a **public subnet**, has an Elastic IP.
- Private subnet route: `0.0.0.0/0 -> nat-xxxx`.
- Allows outbound (updates, API calls), blocks inbound from internet.
- Per-AZ resource: for HA, one per AZ.
- Costs money (hourly + data), a common surprise on the bill. Alternatives: NAT instance, VPC endpoints.

Related: [[07 - NAT]]
