---
tags: [networking, cidr]
---
# CIDR Notation

`IP/prefix` e.g. `10.0.0.0/16`. Prefix = number of fixed network bits.

**Addresses = 2^(32 - prefix)**

| CIDR | Addresses | AWS usable |
|---|---|---|
| /16 | 65,536 | 65,531 |
| /20 | 4,096 | 4,091 |
| /24 | 256 | 251 |
| /28 | 16 | 11 |

- Smaller prefix number = bigger network.
- AWS VPC/subnet sizes: /16 (largest) to /28 (smallest).
- `0.0.0.0/0` = all IPv4; `x.x.x.x/32` = exactly one IP.

Related: [[05 - Subnetting]], [[11 - AWS Subnets]]
