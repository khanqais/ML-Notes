---
tags: [aws, routing, igw]
---
# Route Tables and Internet Gateway

## Route table
- Each subnet is associated with exactly one route table.
- Always has a **local** route for the VPC CIDR (cannot delete).

## Internet Gateway (IGW)
- Horizontally scaled, highly available, free.
- Attached to one VPC.
- Does 1:1 NAT between an instance's private IP and its public IP.
- For internet access, an instance needs: IGW attached + route to IGW + public IP + SG/NACL allowing traffic.

Related: [[06 - Routing and Gateways]]
