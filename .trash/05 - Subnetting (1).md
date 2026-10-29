---
tags: [networking, subnet]
---
# Subnetting

**Subnet** = smaller logical network carved from a bigger one.

## Why
- Isolation and security
- Smaller broadcast domains / organisation
- Different routing per group (public vs private)
- Spread across availability zones

## Example
Split `10.0.0.0/24` into two /25:
- `10.0.0.0/25` -> .0 to .127
- `10.0.0.128/25` -> .128 to .255

Split `10.0.0.0/16` into /24s: 256 subnets (`10.0.0.0/24`, `10.0.1.0/24`, ...).

## Rules
- Subnets must not overlap.
- Each must fit inside the parent block.
- Each subnet has network address (first) and broadcast (last) not usable on classic networks.

Related: [[04 - CIDR Notation]], [[11 - AWS Subnets]]
