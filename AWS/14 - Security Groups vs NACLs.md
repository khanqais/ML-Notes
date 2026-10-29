---
tags: [aws, security]
---
# Security Groups vs NACLs

| | Security Group | NACL |
|---|---|---|
| Level | Instance/ENI | Subnet |
| State | Stateful | Stateless |
| Rules | Allow only | Allow and deny |
| Evaluation | All rules | In number order |
| Default | Deny inbound, allow outbound | Default NACL allows all |

SG can reference another SG (e.g. DB allows 5432 from backend-SG).

Related: [[08 - Ports, Protocols, Firewalls]]
