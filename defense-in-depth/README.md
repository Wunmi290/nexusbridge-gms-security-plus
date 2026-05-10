# SX-02: Defense-in-Depth Security Architecture Document

**Project Title:** Defense-in-Depth Security Architecture for NexusBridge Grant Management System
**System Reference:** NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD
**Framework / Standard:** NIST SP 800-53 Rev 5
**Security Domain:** Threats, Vulnerabilities, and Mitigations
**Author:** Omowumi Adisa, ISSO, NexusBridge Technologies
**Date:** May 2024
**Status:** Complete

## Purpose

Defense-in-depth is the principle that multiple layers of security controls should protect an information system, so that if one layer fails, others remain to prevent compromise. This document describes the defense-in-depth architecture implemented for NexusBridge GMS across five layers: perimeter, network, host, application, and data.

## Layer 1: Perimeter Security

The perimeter is the outermost boundary between NexusBridge GMS and the external internet. NexusBridge GMS uses AWS-native perimeter controls.

| Control | Implementation | NIST Control | Purpose |
|---|---|---|---|
| AWS Shield Standard | Automatically applied to all AWS resources | SC-5 (Denial-of-Service Protection) | Protects against volumetric DDoS attacks at the network edge |
| AWS WAF | Deployed on Application Load Balancer | SI-3 (Malicious Code Protection), SC-7 | Inspects and blocks common web attacks: OWASP Top 10, SQL injection, XSS |
| AWS WAF Managed Rule Sets | AWS Managed Rules: Core Rule Set, Known Bad Inputs | SC-7 | Pre-built rules maintained by AWS threat intelligence |
| Geographic Restriction | WAF geo-block rule restricts access to US locations only | AC-17, SC-7 | Reduces attack surface from non-US threat actors |
| HTTPS-only enforcement | ALB configured to redirect all HTTP to HTTPS (port 443) | SC-8 (Transmission Confidentiality) | Ensures all traffic is encrypted in transit |

**Threat Addressed:** External attackers attempting to reach the application via the public internet through DDoS, web application attacks, or unencrypted connections.

## Layer 2: Network Security

The network layer controls how traffic flows within the AWS VPC and between zones.

| Control | Implementation | NIST Control | Purpose |
|---|---|---|---|
| VPC with Private Subnets | Three-tier subnet architecture: public (ALB only), private app (EC2), private data (RDS) | SC-7 (Boundary Protection) | Ensures application and database tiers are not directly reachable from the internet |
| Security Groups | Stateful firewall rules; app tier allows only port 443 from ALB; RDS allows only port 5432 from app tier | SC-7, AC-3 | Enforces least-privilege network access between layers |
| Network Access Control Lists (NACLs) | Stateless rules at subnet boundary; deny rules for known malicious IP ranges | SC-7 | Provides additional defense at the subnet level |
| VPC Flow Logs | All traffic to/from ENIs logged to S3; analyzed by SIEM | AU-2, AU-12 (Audit Logging) | Provides network visibility for threat detection and forensics |
| NAT Gateway | EC2 instances in private subnets route outbound traffic through NAT | SC-7 | Allows outbound internet access for updates without exposing instances to inbound internet |

**Threat Addressed:** Lateral movement, unauthorized access between network tiers, network reconnaissance.

## Layer 3: Host Security

The host layer secures each EC2 instance and operating system.

| Control | Implementation | NIST Control | Purpose |
|---|---|---|---|
| CIS Benchmarks | Amazon Linux 2023 and Ubuntu 22.04 hardened per CIS Level 1 benchmarks | CM-6 (Configuration Settings) | Reduces attack surface by disabling unnecessary services, setting secure defaults |
| AWS Systems Manager (SSM) | All admin access via SSM Session Manager; no direct SSH; no open port 22 | AC-17 (Remote Access) | Eliminates the most common lateral movement vector (compromised SSH keys) |
| AWS Inspector | Continuous vulnerability scanning for OS and application packages | RA-5 (Vulnerability Scanning) | Identifies new vulnerabilities within 24 hours of detection |
| Endpoint Protection (EDR) | CrowdStrike Falcon agent installed on all EC2 instances | SI-3 (Malicious Code Protection) | Detects and responds to malware, ransomware, and behavioral anomalies at the host level |
| Immutable Infrastructure | EC2 instances replaced rather than patched in place; AMI golden images | CM-2 (Baseline Configuration) | Eliminates configuration drift |
| Instance Metadata Service v2 | IMDSv2 enforced; token-required requests only | AC-3 | Prevents SSRF attacks from exploiting the EC2 metadata endpoint for credential theft |

**Threat Addressed:** Malware, ransomware, unauthorized admin access, configuration drift, SSRF attacks.

## Layer 4: Application Security

The application layer protects the NexusBridge GMS software itself.

| Control | Implementation | NIST Control | Purpose |
|---|---|---|---|
| SAST (Static Analysis) | Semgrep integrated into GitHub CI/CD pipeline; blocks merges with critical findings | SA-11 (Developer Testing) | Catches security vulnerabilities in source code before deployment |
| DAST (Dynamic Analysis) | OWASP ZAP scans run against staging environment before each production release | SA-11 | Identifies runtime vulnerabilities that SAST cannot detect |
| Output Encoding | All user-supplied data encoded before rendering in HTML contexts | SI-10 (Information Input Validation) | Prevents Cross-Site Scripting (XSS) attacks |
| Input Validation | Server-side validation on all input fields; parameterized queries for all database calls | SI-10 | Prevents SQL injection and command injection attacks |
| Session Management | Secure cookie attributes (HttpOnly, Secure, SameSite=Strict); 30-minute timeout | SC-10 (Network Disconnect) | Prevents session hijacking and CSRF attacks |
| Dependency Scanning | Dependabot alerts on GitHub; automated PRs for vulnerable dependencies | SA-12 (Supply Chain) | Detects third-party library vulnerabilities |
| RBAC at Application Layer | Application enforces role-based access; not just infrastructure IAM | AC-3 (Access Enforcement) | Ensures users only see data relevant to their role |

**Threat Addressed:** Injection attacks, XSS, session hijacking, vulnerable dependencies, unauthorized data access.

## Layer 5: Data Security

The data layer protects information at rest and in transit, and governs who can access it.

| Control | Implementation | NIST Control | Purpose |
|---|---|---|---|
| Encryption at Rest | RDS: AES-256 via AWS KMS customer-managed key; S3: SSE-KMS; EBS: encrypted at creation | SC-28 (Protection of Information at Rest) | Renders stolen storage media unreadable without the KMS key |
| Encryption in Transit | TLS 1.2+ enforced on all endpoints; TLS 1.0/1.1 disabled at ALB | SC-8 (Transmission Confidentiality) | Protects data from interception in transit |
| KMS Key Rotation | Automatic annual rotation enabled for all customer-managed KMS keys | SC-12 (Cryptographic Key Management) | Limits the blast radius if a key is ever compromised |
| AWS Macie | Continuously scans S3 buckets for sensitive data; alerts on unexpected PII in unusual locations | SI-12 (Information Management) | Detects accidental exposure or misconfiguration |
| S3 Block Public Access | Organization-level Block Public Access enabled; no S3 bucket can be made public | SC-28, MP-2 | Prevents accidental public exposure of grant data through S3 bucket misconfiguration |
| Database Audit Logging | RDS enhanced monitoring; all queries logged to CloudWatch | AU-2, AU-12 | Provides audit trail for all database access |

**Threat Addressed:** Data theft from storage, interception of data in transit, misconfigured public access, insider data exfiltration.

## Defense-in-Depth Layer Summary

| Layer | Primary Controls | Threats Addressed |
|---|---|---|
| 1. Perimeter | AWS Shield, WAF, HTTPS enforcement, geo-blocking | DDoS, web attacks, unencrypted access |
| 2. Network | VPC segmentation, security groups, NACLs, VPC Flow Logs | Lateral movement, network reconnaissance |
| 3. Host | CIS hardening, SSM (no SSH), Inspector, EDR | Malware, ransomware, config drift |
| 4. Application | SAST, DAST, input validation, session management | Injection, XSS, session hijacking |
| 5. Data | Encryption (rest/transit), Macie, S3 Block Public | Data theft, misconfiguration, insider threat |

## Interview Defense Notes

**What is defense-in-depth and why is it important?**
Defense-in-depth means layering multiple security controls so that if one fails, others remain. No single control is perfect. An attacker who bypasses the perimeter firewall still faces host-level EDR and application-level input validation.

**What is the difference between a security group and a NACL in AWS?**
A security group is stateful: if you allow inbound traffic on a port, the return traffic is automatically allowed. A NACL is stateless: you must explicitly allow both inbound and outbound rules. Security groups operate at the instance level; NACLs operate at the subnet level.

**What is IMDSv2 and why does NexusBridge enforce it?**
IMDSv1 allowed any process on an EC2 instance to query the metadata endpoint (169.254.169.254) and retrieve IAM credentials. An SSRF vulnerability in a web application could be exploited to steal IAM credentials via the metadata service. IMDSv2 requires a session token for metadata requests, blocking SSRF-based credential theft.

---
*Prepared by: Omowumi Adisa, ISSO, NexusBridge Technologies*
*System: NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD*
*[GitHub Portfolio](https://github.com/Wunmi290)*
