# SX-05: GRC Program Overview

**System Reference:** NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD
**Security Domain:** Security Program Management and Oversight
**Framework Alignment:** NIST SP 800-53 Rev 5, FedRAMP Moderate, Privacy Act of 1974, OMB A-130
**Classification:** Controlled Unclassified Information (CUI)

---

## Overview

This document describes the Governance, Risk, and Compliance (GRC) program structure for the NexusBridge Grant Management System. It covers policy governance, risk management methodology, compliance monitoring, and regulatory alignment for a DOE FedRAMP Moderate cloud-hosted system. This artifact demonstrates the ISSO's ability to translate regulatory requirements into operational controls and manage a continuous compliance posture.

---

## Governance Structure

| Role | Individual | GRC Responsibility |
|---|---|---|
| Authorizing Official (AO) | Patricia L. Donovan, DOE CISO | Issue and maintain Authority to Operate (ATO) |
| System Owner | Dr. Marcus T. Webb, DOE Deputy Director | Business risk accountability, mission alignment |
| ISSO | Omowumi Adisa, NexusBridge Technologies | Day-to-day GRC operations, control monitoring |
| SAISO | Gerald F. Munoz, DOE Sr. Agency ISO | Agency-level security policy, oversight |
| Privacy Officer | Helen R. Nguyen, DOE Office of General Counsel | Privacy Act compliance, PIA, SORN |
| 3PAO Lead | Sandra L. Pryce, ClearPoint Assurance LLC | Independent assessment, ConMon validation |
| Cloud Ops Lead | Jerome K. Okafor, NexusBridge Technologies | Technical control implementation |

---

## Acceptable Use Policy (AUP) Summary

**Policy:** NexusBridge GMS Acceptable Use Policy (GMS-AUP-2024-001)
**Authority:** NIST SP 800-53 PL-4, DOE CIO Policy Directive 205.1
**Review Cycle:** Annual (next review: Q1 2025)

**Key AUP Provisions:**

| Provision | Requirement |
|---|---|---|
| System Purpose | GMS may only be used for authorized DOE grant management activities |
| Account Sharing | Prohibited: each user must have a unique, individual account |
| Data Handling | CUI data must remain within approved DOE/GMS systems; no personal cloud storage |
| Remote Access | Requires DOE-approved VPN or AWS direct connect + MFA |
| Password Policy | Minimum 15 characters, complexity required, changed every 60 days |
| Incident Reporting | Users must report suspected incidents within 1 hour to the ISSO |
| Personal Device Use | Prohibited on GMS production environment; BYOD policy applies to email only |
| Software Installation | Prohibited without Cloud Ops and ISSO approval |

**Acknowledgment:** All GMS users must sign AUP acknowledgment annually. Tracked in ServiceNow GMS-AUP-ACK-2024 campaign.

---

## Regulatory Compliance Mapping

| Regulation/Standard | Applicable Requirements | GMS Implementation |
|---|---|---|
| FedRAMP Moderate | 325 controls (NIST SP 800-53 baseline) | SSP, POA&M, ConMon, annual assessment |
| NIST SP 800-53 Rev 5 | 20 control families (AC, AT, AU, CA, CM, CP, IA, IR, MA, MP, PE, PL, PM, PS, PT, RA, SA, SC, SI, SR) | Control implementation per SSP |
| Privacy Act of 1974 | Protect PII collected in federal grant records | PIA, SORN DOE-31, data minimization |
| OMB Circular A-130 | Federal information resources management | Risk management, security planning, authorization |
| OMB M-22-09 (Zero Trust) | Move toward Zero Trust Architecture by FY2024 | MFA enforcement, least privilege, micro-segmentation roadmap |
| FISMA 2014 | Annual FISMA reporting to OMB | POA&M, ConMon metrics, CIO dashboard reporting |
| DOE Order 205.1C | Departmental Cybersecurity Program | Compliance with DOE-specific security requirements |

---

## Risk Assessment Summary

**Methodology:** NIST SP 800-30 Rev 1 (Guide for Conducting Risk Assessments)
**Last Assessment:** Q3 2024 | **Next Assessment:** Q3 2025

| Risk ID | Threat | Vulnerability | Likelihood | Impact | Risk Level | Treatment |
|---|---|---|---|---|---|---|
| R-001 | Phishing attack | User susceptibility to social engineering | High | High | Critical | User awareness training, MFA enforcement, email filtering |
| R-002 | Unauthorized access | Weak IAM controls | Medium | High | High | RBAC implementation, quarterly access reviews |
| R-003 | Data exfiltration | Misconfigured S3 bucket permissions | Low | Critical | High | S3 block public access, bucket policy audit |
| R-004 | Ransomware | Unpatched vulnerabilities | Medium | High | High | 30-day patch SLA, Nessus scanning, immutable backups |
| R-005 | Insider threat | Excessive privilege, lack of monitoring | Low | High | Medium | Least privilege, CloudTrail logging, anomaly detection |
| R-006 | Third-party supply chain compromise | Unvetted software dependencies | Low | Medium | Medium | Software composition analysis, vendor risk assessment |
| R-007 | Cloud service outage | Single-region dependency | Low | High | Medium | Multi-AZ deployment, RTO/RPO-tested backup procedures |

---

## Compliance Dashboard (Q3 2024)

| Domain | Control Family | Controls Implemented | Controls in POA&M | Compliance Rate |
|---|---|---|---|---|
| Access Control | AC | 22 / 22 | 0 | 100% |
| Audit and Accountability | AU | 15 / 16 | 1 (AU-6 enhancement) | 94% |
| Configuration Management | CM | 12 / 12 | 0 | 100% |
| Identification and Authentication | IA | 12 / 12 | 0 | 100% |
| Incident Response | IR | 10 / 10 | 0 | 100% |
| Risk Assessment | RA | 9 / 10 | 1 (RA-5 scan frequency) | 90% |
| System and Communications Protection | SC | 28 / 30 | 2 (TLS config updates) | 93% |
| System and Information Integrity | SI | 16 / 17 | 1 (SI-4 enhancement) | 94% |
| **Overall** | **All Families** | **186 / 193** | **7** | **96.4%** |

**Open POA&M Items:** 7 (all Moderate risk or below, on track for remediation)
**ATO Status:** Active | **ATO Expiration:** December 2025 | **Next Annual Assessment:** Q4 2025

---

## Privacy Program Integration

**Privacy Act Applicability:** GMS collects and processes PII from federal grant applicants and awardees, including name, SSN (partial), employer, financial data, and research affiliation. This data is covered under System of Records Notice (SORN) DOE-31, published in the Federal Register.

**Privacy Controls (NIST SP 800-53 PT Family):**

| Control | Title | Implementation |
|---|---|---|
| PT-1 | Privacy Policy and Procedures | DOE Privacy Program Policy v4.0 |
| PT-2 | Authority to Process PII | Privacy Act authority documented in PIA |
| PT-3 | Purpose Specification | PII collected only for grant management purposes |
| PT-5 | Privacy Notice | Privacy notice displayed at GMS portal login |
| PT-6 | System of Records Notice | SORN DOE-31 published, updated annually |
| PT-7 | Specific Categories of PII | SSN handling restricted; access limited to authorized roles |

**Privacy Impact Assessment (PIA):** Completed Q1 2024. Next review triggered by any system change that affects PII collection, use, or sharing. Privacy Officer Helen R. Nguyen coordinates with ISSO on all PIA updates.

---

## Interview Defense Notes

**Q: What is GRC and how does an ISSO contribute to it?**

A: GRC stands for Governance, Risk, and Compliance. Governance is the policies and decision-making structure: who owns risk, who approves access. Risk is the identification and treatment of threats to the system. Compliance is demonstrating that controls meet regulatory requirements. As ISSO, I sit at the intersection: I translate policies into controls, track risks in the POA&M, and produce evidence for compliance reviews and the annual FedRAMP assessment.

**Q: What is the difference between a risk assessment and a vulnerability assessment?**

A: A vulnerability assessment (like a Nessus scan) identifies technical weaknesses in systems. A risk assessment (NIST SP 800-30) is broader: it evaluates threat actors, threat events, vulnerabilities, likelihood, impact, and recommends treatment. Vulnerability scan findings feed into the risk register: a critical finding becomes a High risk item in the POA&M with a remediation timeline.

**Q: How does the Privacy Act relate to cybersecurity?**

A: The Privacy Act of 1974 governs how federal agencies collect, maintain, and use personally identifiable information (PII). For GMS, this means we must publish a System of Records Notice (SORN), complete a Privacy Impact Assessment (PIA), and ensure PII is only used for the purpose it was collected. From a security standpoint, this drives access controls (only authorized roles can access SSN fields), audit logging (all PII access logged), and data minimization (only collect what is necessary).

**Q: What is a POA&M and how do you manage it?**

A: A Plan of Action and Milestones (POA&M) is a formal document that tracks identified security weaknesses, their risk level, remediation owner, and target completion date. I update the GMS POA&M monthly as part of ConMon. Each item has a weakness description, root cause, risk rating, proposed mitigation, and milestone dates. The ATO condition requires all Critical and High POA&M items to be remediated within 30 days. I report POA&M status to the AO and 3PAO quarterly.

**Q: What does FedRAMP ConMon involve?**

A: Continuous Monitoring (ConMon) for FedRAMP includes: monthly vulnerability scans (Nessus) with results uploaded to the FedRAMP repository; monthly POA&M updates; OS/software patch compliance reporting; incident reporting per US-CERT SLAs; annual penetration testing; and quarterly access reviews. I coordinate all of this as ISSO and deliver monthly ConMon packages to the 3PAO (Sandra L. Pryce, ClearPoint Assurance) and the DOE AO.

---

*Prepared by: Omowumi Adisa, ISSO, NexusBridge Technologies*
*System: NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD*
*[GitHub Portfolio](https://github.com/Wunmi290)*
