---
tags: [interview, aws]
---
# Interview Q&A

**Public vs private subnet?** Public has a route to an IGW; private does not.

**How does a private instance reach the internet?** Via a NAT gateway in a public subnet.

**SG vs NACL?** SG: instance level, stateful, allow-only. NACL: subnet level, stateless, allow and deny.

**Can a subnet span AZs?** No, one AZ only.

**Why does AWS reserve 5 IPs?** Network address, VPC router, DNS, future use, broadcast.

**How many IPs in /24 on AWS?** 256 total, 251 usable.

**Is VPC peering transitive?** No.

**Why can't I reach my EC2?** Check: public IP, IGW attached, route table, SG, NACL, OS firewall.

**Design a secure web app VPC.** See [[16 - Three-Tier VPC Design]].
