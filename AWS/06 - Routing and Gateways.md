---
tags: [networking, routing]
---
# Routing and Gateways

- **Routing table**: list of `destination CIDR -> target`.
- **Default gateway**: where traffic goes when no specific route matches.
- **Longest prefix match**: the most specific route wins. `10.0.1.0/24` beats `10.0.0.0/16` beats `0.0.0.0/0`.

Example:
| Destination | Target |
|---|---|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | igw-123 |

Packet to `10.0.5.9` -> local. Packet to `8.8.8.8` -> internet gateway.

Related: [[12 - Route Tables and Internet Gateway]]
