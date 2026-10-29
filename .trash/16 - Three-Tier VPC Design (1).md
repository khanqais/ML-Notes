---
tags: [aws, design]
---
# Three-Tier VPC Design

VPC `10.0.0.0/16`
| Subnet | AZ | Type | Holds |
|---|---|---|---|
| 10.0.1.0/24 | a | Public | ALB, NAT GW |
| 10.0.2.0/24 | b | Public | ALB, NAT GW |
| 10.0.11.0/24 | a | Private | Backend (Express/Go) |
| 10.0.12.0/24 | b | Private | Backend |
| 10.0.21.0/24 | a | Private | RDS |
| 10.0.22.0/24 | b | Private | RDS |

Flow: Internet -> IGW -> ALB -> backend -> DB.
SG chain: ALB-SG (443 from anywhere) -> App-SG (app port from ALB-SG) -> DB-SG (5432 from App-SG).

Tips: plan CIDR so it does not overlap with other VPCs or on-prem if you will peer.
