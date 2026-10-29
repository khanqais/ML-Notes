---
tags: [networking, basics]
---
# Networking Basics

- **Network**: devices that can talk to each other.
- **Host**: any device with an IP (laptop, server, EC2).
- **Packet**: small chunk of data with a source IP and a destination IP.
- **Router**: forwards packets between networks.
- **Switch**: connects devices inside one network.
- **LAN / WAN**: local network vs wide network (internet).
- **Client-server**: client sends request, server responds.

## OSI / TCP-IP layers (what matters)
| Layer | Example | VPC relevance |
|---|---|---|
| L3 Network | IP, routing | CIDR, route tables, subnets |
| L4 Transport | TCP, UDP, ports | Security groups, NACLs |
| L7 Application | HTTP, DNS | ALB, Route 53 |

Next: [[02 - IP Addresses]]
