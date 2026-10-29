---
tags: [networking, binary]
---
# Binary and Subnet Mask

`172.31.0.128` in binary: `10101100.00011111.00000000.10000000`

## Subnet mask
Tells which bits are network and which are host.
- `255.255.0.0` = `/16` (16 network bits)
- `255.255.240.0` = `/20`
- `255.255.255.0` = `/24`

## Network ID
IP AND mask = network address.
Example: `172.31.1.27` with `/20` -> network `172.31.0.0`.

Why? /20 means the third octet is split: first 4 bits network, last 4 bits host. 1 is in the 0-15 block, so the network is 172.31.0.0.

## Block size trick
Block size = 256 - mask octet. For /20, third octet mask 240, so block = 16: subnets start at 0, 16, 32 ... in the third octet.

Next: [[04 - CIDR Notation]]
