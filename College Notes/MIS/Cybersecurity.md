Identify measures to improve cyber-security with examples. Give me a full answer, and in that, can I write cross- cross-site scripting?Um, CSFR?Etcetera, and all those security points.Also.

Yes—you **can and should** include **XSS (Cross-Site Scripting)** and **CSRF (Cross-Site Request Forgery)** in this answer, but only as *specific technical measures/examples* under web-application security. Do not make the whole answer only about XSS and CSRF; the examiner expects broad cyber-security measures such as MFA, firewalls, encryption, patching, backups, and employee training. NIST recommends managing access, training users, updating software, scanning vulnerabilities, and maintaining backups as core protective practices.[1]

## Full 10-mark answer

### Introduction

Cyber-security means protecting an organization’s computers, networks, applications, data, and users from unauthorized access, misuse, modification, disruption, or destruction. It aims to maintain the **CIA triad**: Confidentiality, Integrity, and Availability.

Cyber-security cannot depend on one single control. An organization must use a combination of technical, administrative, and physical measures. The NIST Cybersecurity Framework organizes this protection into functions such as Identify, Protect, Detect, Respond, and Recover.[2][3]

### 1. Use strong passwords and multi-factor authentication

Organizations should require strong, unique passwords and implement Multi-Factor Authentication (MFA). MFA requires a user to provide two or more verification factors, such as a password and a one-time code sent to a mobile phone.

**Example:** A bank customer enters their password and then approves a login request through an OTP or authenticator app. Even if an attacker steals the password, they cannot log in without the second factor.

NIST specifically recommends authenticating users before granting access to information systems and applications.[1]

### 2. Apply the principle of least privilege

Employees should be given only the minimum access required to perform their jobs. This is called the **principle of least privilege**.

**Example:** A sales employee may access customer contact details but should not have permission to change salary records or delete financial transactions.

This reduces the damage if an employee account is compromised. NIST recommends unique user accounts and access limited to necessary resources.[1]

### 3. Install firewalls and antivirus software

A firewall monitors and controls incoming and outgoing network traffic based on security rules. Antivirus and anti-malware software detect and remove malicious programs such as viruses, worms, trojans, ransomware, and spyware.

**Example:** A company firewall can block traffic from suspicious IP addresses, while antivirus software detects malware downloaded through an infected email attachment.

Endpoint security products and controlled device configurations are recommended protective measures.[1]

### 4. Keep software updated and patch vulnerabilities

Software updates and security patches fix known weaknesses in operating systems, applications, browsers, and network devices. Attackers often exploit unpatched systems.

**Example:** If a company’s web server has a known vulnerability, applying the vendor’s security patch prevents attackers from exploiting it.

NIST recommends enabling automatic updates, replacing unsupported end-of-life software, and scanning systems for vulnerabilities.[1]

### 5. Encrypt sensitive data

Encryption converts readable data into unreadable coded form. Only authorized users with the correct decryption key can read it.

**Example:** An e-commerce website uses HTTPS/SSL encryption so that a customer’s credit-card details cannot be read while being transmitted over the internet.

Organizations should encrypt data both during transmission and when stored on servers, laptops, or mobile devices.

### 6. Take regular backups

Regular backups protect data from ransomware, hardware failure, accidental deletion, and cyber-attacks. Backups should be tested and at least one copy should be kept offline or isolated from the main network.

**Example:** If ransomware encrypts a hospital’s patient records, the hospital can restore clean backup data instead of paying the attacker.

NIST recommends regular backups, including at least one frequently updated offline backup to protect against ransomware.[1]

### 7. Conduct employee cyber-security training

Employees are often the weakest link in cyber-security because attackers use phishing emails, fake links, and social engineering to trick them. Regular training teaches employees how to identify suspicious emails, avoid unknown attachments, and report incidents.

**Example:** An employee receives an email pretending to be from the IT department asking for their password. After training, the employee recognizes it as phishing and reports it instead of sharing credentials.

NIST recommends regular user training on cyber-security policies and common attacks.[1]

### 8. Use web-application security controls

For websites and online applications, organizations must protect against attacks such as SQL injection, XSS, and CSRF.

#### Cross-Site Scripting (XSS)

**XSS** is an attack in which an attacker injects malicious JavaScript or HTML into a trusted website. When another user views the affected page, the malicious script runs in that user’s browser and may steal session cookies or personal information.[4][5]

**Example:** An attacker submits a product review containing malicious JavaScript. If the website displays the review without proper validation, every visitor’s browser may execute the script.

**Measures to prevent XSS:**

- Validate and filter user input.
- Encode output before displaying user-supplied data.
- Use secure coding frameworks and safe APIs.
- Apply Content Security Policy (CSP).
- Use HttpOnly and Secure cookies so JavaScript cannot easily access session cookies.
- Conduct regular security testing and code review.[4][6]

#### Cross-Site Request Forgery (CSRF)

**CSRF** is an attack in which a malicious website, email, or message tricks an already logged-in user’s browser into sending an unwanted request to a trusted website. Because the browser automatically sends the user’s session cookie, the website may treat the request as genuine.[7][8]

**Example:** A user is logged into an online banking website. The user opens a malicious email or website that secretly sends a fund-transfer request to the bank. If CSRF protection is absent, the bank may process the request using the user’s active session.

**Measures to prevent CSRF:**

- Use anti-CSRF tokens in forms and validate them on the server.
- Use SameSite cookies to restrict cookies from being sent in cross-site requests.
- Verify the request origin or referrer.
- Require re-authentication for sensitive actions such as changing a password or transferring money.
- Use secure frameworks with built-in CSRF protection.[7][9]

**Important exam point:** XSS injects malicious script into a web page, whereas CSRF tricks an authenticated user’s browser into performing an unwanted action.[6]

### 9. Monitor networks and detect incidents

Organizations should continuously monitor systems, networks, and user activity to identify unusual behaviour. Tools such as intrusion detection systems (IDS), intrusion prevention systems (IPS), Security Information and Event Management (SIEM), and log monitoring help detect attacks early.

**Example:** If an employee account suddenly downloads a very large amount of data at midnight from an unusual location, the system can generate an alert for the security team.

The Detect function of the NIST framework focuses on identifying cybersecurity events through continuous monitoring and analysis of anomalies.[3][10]

### 10. Prepare an incident-response and recovery plan

Even strong security controls cannot guarantee that an attack will never happen. Therefore, organizations need an incident-response plan that explains how to identify, contain, investigate, communicate, and recover from a cyber-attack.

**Example:** If ransomware affects a company’s servers, the incident-response team should isolate infected systems, restore data from backups, change compromised passwords, inform affected users if required, and identify the cause of the attack.

The Respond and Recover functions of the NIST framework focus on taking action against detected incidents and restoring impaired services.[3][10]

## Summary table

| Measure | Purpose | Example |
|---|---|---|
| Strong passwords and MFA | Prevent unauthorized login | OTP after password |
| Least privilege | Limit employee access | Sales staff cannot access payroll |
| Firewall and antivirus | Block malicious traffic and malware | Firewall blocks suspicious IP |
| Patch management | Fix software weaknesses | Update web-server vulnerability |
| Encryption | Protect confidential data | HTTPS for online payments |
| Backup | Recover lost or encrypted data | Restore files after ransomware |
| Employee training | Prevent phishing and social engineering | Report fake IT email |
| Input validation and output encoding | Prevent XSS | Encode user comments |
| CSRF tokens and SameSite cookies | Prevent CSRF | Token required for money transfer |
| Monitoring and SIEM | Detect abnormal activity | Alert for unusual data download |
| Incident-response plan | Handle and recover from attacks | Isolate infected system and restore backup |

## Conclusion

Cyber-security should be implemented through a **layered defence** approach. No single measure is sufficient; organizations must combine access control, MFA, firewalls, patching, encryption, backups, employee awareness, secure coding, monitoring, and incident-response planning. For web applications, XSS and CSRF protections are important technical measures, but they must be used along with broader organizational security controls.[1][4][7]

