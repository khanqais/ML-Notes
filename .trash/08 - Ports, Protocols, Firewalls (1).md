---
tags: [networking, security]
---
# Ports, Protocols, Firewalls

- **TCP**: reliable, connection oriented (HTTP, SSH).
- **UDP**: fast, no guarantee (DNS, streaming).
- **ICMP**: ping.
- **Port**: identifies a service on a host. 22 SSH, 80 HTTP, 443 HTTPS, 5432 PostgreSQL, 3306 MySQL, 6379 Redis, 27017 MongoDB.
- **DNS**: name -> IP.
- **Firewall rule** = protocol + port + source/destination.
- **Stateful** firewall remembers connections (reply automatically allowed). **Stateless** needs rules both ways.

Related: [[14 - Security Groups vs NACLs]]
