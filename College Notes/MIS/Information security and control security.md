https://www.geeksforgeeks.org/computer-networks/what-is-information-security/

Give an understanding of types of controls to achieve security. For this.

Security controls are the safeguards used to protect information systems, data, networks, and physical assets from threats. They are mainly classified as **administrative, technical, and physical controls**; they can also be classified by function as **preventive, detective, corrective, deterrent, recovery, and compensating controls**. NIST defines security controls as management, operational, and technical safeguards that protect the confidentiality, integrity, and availability of information systems.[1][2]

## Full 10-mark answer

### Introduction

An organization cannot achieve information security using only one control. It must use a combination of policies, technology, and physical protection. These controls reduce risks such as unauthorized access, data theft, malware, system failure, insider misuse, and cyber-attacks.

The main objective of security controls is to maintain the **CIA triad**:

- **Confidentiality:** Only authorized people can access information.
- **Integrity:** Information is accurate and cannot be improperly changed.
- **Availability:** Authorized users can access information when required.

Security controls are commonly grouped into three major categories: **administrative controls, technical controls, and physical controls**.[2][3]

## 1. Administrative controls

Administrative controls are rules, policies, procedures, and practices created by management to guide employee behaviour and manage security risk. They are also called **management controls** or **organizational controls**.[3][4]

### Examples

- Security policy: Rules for acceptable use of company computers, email, internet, and mobile devices.
- Password policy: Minimum password length, complexity, expiry, and prohibition of password sharing.
- Access-control policy: Employees receive access only according to their job role.
- Security awareness training: Employees are trained to identify phishing emails and social-engineering attacks.
- Background verification: Checking employee history before giving access to sensitive systems.
- Risk assessment: Identifying threats, vulnerabilities, and possible impact on information assets.
- Incident-response policy: Steps to follow when a cyber-attack or data breach occurs.
- Audit and compliance review: Checking whether employees and systems follow security rules.

**Example:** A company policy may state that every employee must use MFA, must not share passwords, and must report suspicious emails to the IT department. This is an administrative control because it guides human behaviour.[3][5]

## 2. Technical controls

Technical controls are security safeguards implemented through hardware, software, firmware, or network technology. They are also called **logical controls** because they use technology to protect systems and data.[2][3]

### Examples

- **Firewall:** Controls incoming and outgoing network traffic.
- **Antivirus and anti-malware software:** Detects and removes malicious programs.
- **Encryption:** Protects data during storage and transmission.
- **Multi-factor authentication (MFA):** Requires a password plus OTP, biometric verification, or authenticator app.
- **Access-control lists (ACLs):** Define which users can access files, folders, or systems.
- **Intrusion Detection System (IDS):** Detects suspicious network activity.
- **Intrusion Prevention System (IPS):** Detects and blocks malicious activity.
- **Patch management:** Updates software to remove known vulnerabilities.
- **SIEM:** Collects and analyses security logs from different systems.
- **Network segmentation:** Divides a network into smaller parts to limit attacker movement.

**Example:** A bank uses encryption for online transactions, MFA for customer login, and a firewall to block unauthorized network traffic. These are technical controls because they are enforced by technology.[2][6]

## 3. Physical controls

Physical controls protect the physical environment where computers, servers, network devices, and storage media are located. They prevent unauthorized physical access, theft, damage, fire, and environmental hazards.[3][7]

### Examples

- Locks, keys, and security doors.
- Fences, gates, and security guards.
- CCTV cameras and surveillance systems.
- ID cards, access badges, and biometric scanners.
- Fire extinguishers, smoke detectors, and fire-suppression systems.
- Air conditioning and humidity control for server rooms.
- UPS and backup generators.
- Secure disposal of old hard disks and documents.

**Example:** A company server room may have biometric access, CCTV monitoring, fire extinguishers, and restricted entry for authorized staff only. These controls prevent physical theft or damage to servers.[6][7]

## Classification by function

Security controls can also be classified according to **when they act** in relation to a security incident.

| Type of control | Purpose | Examples | Example scenario |
|---|---|---|---|
| Preventive control | Stops an attack before it occurs | MFA, firewall, encryption, access control, training | MFA prevents an attacker from logging in using a stolen password |
| Detective control | Identifies that an attack or violation has occurred | IDS, CCTV, audit logs, SIEM, security audit | SIEM alerts the security team after unusual login activity |
| Corrective control | Fixes the damage after an incident | Patching, restoring backup, reimaging device | Company restores systems from backup after ransomware |
| Deterrent control | Discourages attackers or employees from violating rules | Warning signs, CCTV, security policy, penalties | CCTV discourages unauthorized entry |
| Recovery control | Restores normal operations after disruption | Backup, disaster-recovery site, failover system | Company switches to backup server after main server failure |
| Compensating control | Alternative control when the primary control is not possible | Extra monitoring when MFA cannot be used | Additional logging used where legacy system cannot support MFA |

NIST describes preventive controls as controls that inhibit policy violations, while detective controls warn of violations or attempted violations through mechanisms such as audit trails, intrusion detection, and checksums.[2]

## Preventive controls

Preventive controls are designed to stop a security incident before it happens. They are the first line of defence.

Examples include:

- Strong passwords and MFA.
- Firewalls.
- Encryption.
- Role-based access control.
- Employee security training.
- Software patching.
- Physical locks and biometric access.

**Example:** A company gives a finance officer access only to accounting software, not to HR salary records. This prevents unauthorized access.

## Detective controls

Detective controls identify security incidents during or after they occur. They do not necessarily stop the attack, but they help the organization discover it quickly.

Examples include:

- Audit trails and system logs.
- Intrusion Detection Systems.
- CCTV cameras.
- SIEM alerts.
- File-integrity monitoring.
- Security audits.
- Network traffic monitoring.

**Example:** If an employee downloads a large amount of confidential data at an unusual time, log monitoring can generate an alert for the security team.

## Corrective controls

Corrective controls reduce the impact of an incident after it has occurred and help restore normal operations.

Examples include:

- Applying security patches after discovering a vulnerability.
- Restoring data from backup.
- Removing malware from infected systems.
- Resetting compromised passwords.
- Replacing damaged hardware.
- Blocking a compromised user account.

**Example:** After ransomware attacks a company’s server, the IT team restores clean data from an offline backup and patches the vulnerability that allowed the attack.

## Recovery controls

Recovery controls help an organization return to normal business operations after a major disruption such as a cyber-attack, fire, flood, or system failure.

Examples include:

- Regular backups.
- Disaster-recovery plan.
- Alternate data centre.
- Cloud failover.
- Business-continuity plan.

**Example:** If a company’s main data centre is destroyed by fire, it can shift operations to a backup data centre or cloud environment.

## Deterrent controls

Deterrent controls discourage people from attempting unauthorized actions. They may not physically stop an attack, but they reduce the likelihood of an attack.

Examples include:

- Warning signs such as “CCTV surveillance in operation.”
- Security policies stating penalties for misuse.
- Visible security guards.
- Login-failure warnings.
- Legal agreements and non-disclosure agreements.

**Example:** A warning message on a login page stating that unauthorized access is monitored and legally punishable can discourage attackers.

## Compensating controls

Compensating controls are alternative safeguards used when the main control cannot be implemented or is not fully effective.

**Example:** If an old legacy application cannot support MFA, the organization may use strict IP restrictions, network segmentation, additional logging, and frequent monitoring as compensating controls.

## Relationship between controls

| Control type | Main role | Example |
|---|---|---|
| Administrative | Directs people and processes | Security policy, training |
| Technical | Protects systems using technology | Firewall, encryption, MFA |
| Physical | Protects physical assets | Locks, CCTV, biometric access |
| Preventive | Stops incidents | MFA, firewall |
| Detective | Identifies incidents | IDS, audit logs |
| Corrective | Repairs damage | Backup restoration, patching |
| Recovery | Restores operations | Disaster-recovery plan |
| Deterrent | Discourages attacks | CCTV, warning notices |
| Compensating | Provides alternative protection | Extra monitoring for legacy systems |

## Conclusion

To achieve information security, an organization must use a **layered defence** approach. Administrative controls guide people, technical controls protect systems and data, and physical controls protect hardware and facilities. In addition, preventive, detective, corrective, recovery, deterrent, and compensating controls must work together to reduce security risks and maintain confidentiality, integrity, and availability.[1][2][3]

## What to memorize

Remember these **three main categories**:

1. Administrative controls: policies, training, access rules.  
2. Technical controls: firewall, encryption, MFA, antivirus.  
3. Physical controls: locks, CCTV, biometrics, guards.  

And remember these **six functional types**:

1. Preventive  
2. Detective  
3. Corrective  
4. Recovery  
5. Deterrent  
6. Compensating  

For a 10-mark answer, write the introduction, CIA triad, three main categories with examples, then the functional classification table. This structure will give you enough points and make the answer look complete.[2][3][6]

Citations:
[1] NISTIR 7359, Information Security Guide For Government ... https://nvlpubs.nist.gov/nistpubs/legacy/ir/nistir7359.pdf
[2] Risk Management Guide for Information Technology Systems https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=151254
[3] Types of Security Controls With Examples [How to Implement] https://sprinto.com/blog/types-of-security-controls/
[4] Types of Security Controls: Administrative, Physical, Technical & More https://www.infosectrain.com/blog/types-of-security-controls
[5] Understanding the Types of Security Controls and Why They Matter https://spog.ai/blog/understanding-the-different-types-of-security-controls-and-why-they-matter/
[6] The 3 Types Of Security Controls (Expert Explains) https://purplesec.us/learn/security-controls/
[7] Guide to Operational Technology (OT) Security https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf
[8] Downloads - NIST Risk Management Framework | CSRC https://csrc.nist.gov/projects/risk-management/sp800-53-controls/downloads
[9] An Introduction to Information Security - NIST https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-12r1.pdf
[10] Guide to Information Technology Security Services - NIST https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-35.pdf
[11] Understanding the three main types of security controls - 6Clicks https://www.6clicks.com/resources/blog/types-of-security-controls
[12] Understanding Security Control Categories: Technical, Operational, Managerial, and ... https://www.trainingtraining.training/blog/understanding-security-control-categories
[13] Understanding the three categories of security controls https://www.linkedin.com/posts/manish2sh_categories-of-security-controls-in-cybersecurity-activity-7363861746324828160-yK_4
[14] Network Security https://www.geeksforgeeks.org/computer-networks/network-security/
[15] NIST SP 800-12: Chapter 1 Introduction https://csrc.nist.rip/publications/nistpubs/800-12/800-12-html/chapter1.html
