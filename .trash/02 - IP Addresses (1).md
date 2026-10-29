---
tags: [networking, ip]
---
# IP Addresses

An IP address identifies a device on a network.

## IPv4
- 32 bits, 4 octets, each 0-255. Example: `172.31.0.128`.
- Total about 4.3 billion addresses, so we ran short, hence NAT and IPv6.
- Two parts: **network part** + **host part**.

## Private ranges (RFC 1918)
| Range | CIDR |
|---|---|
| 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 |
| 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 |
| 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 |

Private IPs are not routable on the public internet.

## Public IP
Routable on the internet, unique globally. In AWS, comes from Amazon's pool (or an Elastic IP).

## IPv6
128 bits, written in hex, huge address space. Mostly no NAT needed.

## Special addresses
- `127.0.0.1` = localhost
- `0.0.0.0/0` = "every address" (used as the default route)

Link: [[03 - Binary and Subnet Mask]], [[07 - NAT]]
