# SX-04a: Incident Response Playbook: Phishing Attack

**System Reference:** NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD
**CompTIA Security+ Domain 4:** Security Operations
**Framework Alignment:** NIST SP 800-61 Rev 2, NIST SP 800-53 IR Family
**Classification:** Controlled Unclassified Information (CUI)

---

## Overview

This playbook documents the incident response (IR) procedures for phishing attacks targeting the NexusBridge Grant Management System. It maps to NIST SP 800-61 Rev 2 lifecycle phases and FedRAMP Moderate IR controls. This artifact supports the ISSO role by demonstrating structured incident handling capability aligned to federal cybersecurity requirements.

---

## NIST SP 800-61 Rev 2 IR Lifecycle

| Phase | Description | Key Activities |
|---|---|---|
| Preparation | Establish IR capability | IR policy, playbooks, training, contact list |
| Detection and Analysis | Identify and confirm incident | Alert triage, log review, severity scoring |
| Containment, Eradication, Recovery | Limit damage, remove threat, restore | Isolate systems, remove malware, restore from backup |
| Post-Incident Activity | Learn and improve | Lessons learned, metrics, report to AO |

---

## Phishing-Specific Detection Sources

| Source | Tool/Method | Indicator |
|---|---|---|
| Email gateway | Proofpoint / Microsoft Defender | Suspicious sender, spoofed domain |
| SIEM alerts | Splunk | Spike in failed login attempts post-email |
| User reports | ServiceNow ticket | Employee reports suspicious link/attachment |
| Endpoint telemetry | CrowdStrike / Defender ATP | Macro execution, powershell spawn |
| DNS logs | AWS Route 53 / VPC Flow Logs | Resolution of known malicious domain |

---

## Incident Severity Classification

| Severity | Criteria | Response Time | Notification |
|---|---|---|---|
| Critical (P1) | Credential compromise + data exfiltration confirmed | 1 hour | AO, SAISO, CISA, US-CERT |
| High (P2) | Credential compromise, no confirmed exfil | 4 hours | AO, SAISO, ISSO |
| Medium (P3) | Phishing email clicked, no credential entry | 24 hours | ISSO, Cloud Ops Lead |
| Low (P4) | Phishing email received, not clicked | 72 hours | ISSO log only |

---

## Containment Steps (P2/P3 Scenario)

**Immediate Actions (0-4 hours):**

1. Disable compromised user account in AWS IAM and DOE AD
2. Revoke active session tokens (AWS SSO, NexusBridge GMS portal)
3. Block malicious sender domain at email gateway
4. Quarantine endpoint via CrowdStrike remote containment
5. Preserve memory dump and disk image for forensic analysis
6. Notify ISSO (Omowumi Adisa) and Cloud Ops Lead (Jerome K. Okafor)

**Short-Term Actions (4-24 hours):**

7. Conduct log review across Splunk for lateral movement indicators
8. Check AWS CloudTrail for unauthorized API calls from compromised account
9. Review VPC Flow Logs for anomalous outbound connections
10. Scan all endpoints that received the same phishing email (Nessus)
11. Draft incident report draft for AO (Patricia L. Donovan) review

---

## Eradication and Recovery

**Eradication:**

- Remove malicious email from all mailboxes using admin purge (Microsoft 365 Compliance)
- Reset credentials for all affected accounts (enforce MFA re-enrollment)
- Patch exploited vulnerability (if applicable)
- Update endpoint AV/EDR signatures
- Block IOCs (IP addresses, domains, file hashes) at WAF and DNS resolver

**Recovery:**

- Re-enable user account after credential reset and MFA confirmation
- Restore affected GMS data from AWS S3 versioned backup if data integrity compromised
- Conduct smoke testing on NexusBridge GMS portal to confirm normal operation
- Resume normal operations with enhanced monitoring for 30 days

---

## Post-Incident Activity

**Lessons Learned Meeting (within 2 weeks):**

- Attendees: Omowumi Adisa (ISSO), Jerome K. Okafor (Cloud Ops), Helen R. Nguyen (Privacy Officer), DOE SAISO Gerald F. Munoz
- Review: Detection timeline, containment effectiveness, user training gaps
- Output: Updated playbook, POA&M entry if control gap identified

**Required Reporting:**

| Report | Recipient | Deadline |
|---|---|---|
| Initial IR Notification | AO Patricia L. Donovan, DOE SAISO | Within 1 hour (P1/P2) |
| US-CERT Incident Report | CISA US-CERT portal | Within 1 hour (P1) / 24 hours (P2) |
| Internal Incident Report | DOE CISO office | Within 72 hours |
| Privacy Breach Notification | Helen R. Nguyen, Privacy Officer | Within 24 hours if PII involved |
| FedRAMP ConMon Report | 3PAO Sandra L. Pryce | Monthly ConMon update |

---

## Key Performance Metrics

| Metric | Target | Source |
|---|---|---|
| Mean Time to Detect (MTTD) | < 4 hours | Splunk alert timestamps |
| Mean Time to Contain (MTTC) | < 8 hours | IR ticket open/contain timestamps |
| Mean Time to Recover (MTTR) | < 24 hours | ServiceNow ITSM |
| Incident Recurrence Rate | < 5% | Quarterly IR trend report |
| User Phishing Simulation Click Rate | < 10% | KnowBe4 / Proofpoint Awareness |

---

## NIST SP 800-53 Control Mapping

| Control | Title | Implementation |
|---|---|---|
| IR-1 | Incident Response Policy | DOE IR Policy v3.1, NexusBridge IR-SOP-001 |
| IR-2 | Incident Response Training | Annual IR training, quarterly tabletop exercises |
| IR-4 | Incident Handling | This playbook: 4-phase NIST lifecycle |
| IR-5 | Incident Monitoring | Splunk SIEM, AWS GuardDuty, CloudTrail |
| IR-6 | Incident Reporting | US-CERT reporting within defined SLAs |
| IR-8 | Incident Response Plan | NexusBridge GMS IRP v2.0, reviewed annually |
| SI-3 | Malicious Code Protection | CrowdStrike EDR, email gateway AV scanning |
| AC-2 | Account Management | IAM disable/revoke procedures documented above |

---

## Interview Defense Notes

**Q: Walk me through how you would handle a phishing incident.**

A: I follow the NIST SP 800-61 Rev 2 lifecycle. First, during Preparation, I ensure we have a current playbook, trained staff, and tested communication trees. Upon Detection, I triage the alert in Splunk, confirm the phishing email via the email gateway, and assign a severity. For a P2 incident (credential compromise, no confirmed exfil), I immediately disable the account in IAM, revoke session tokens, quarantine the endpoint, and notify the AO within 4 hours. During Recovery, I reset credentials, enforce MFA re-enrollment, and restore from backup if needed. Post-Incident, I lead a lessons learned meeting and update our POA&M if a control gap is found.

**Q: What is the difference between containment and eradication?**

A: Containment limits the spread of the incident without fully resolving it: isolating a system, disabling an account. Eradication removes the root cause: deleting malware, patching the vulnerability, revoking the phishing email. You contain first to preserve forensic evidence, then eradicate once you understand the full scope.

**Q: What is US-CERT and when do you report to them?**

A: US-CERT (United States Computer Emergency Readiness Team), now part of CISA, receives federal incident reports. For FedRAMP systems, we report within 1 hour for critical incidents (P1) and within 24 hours for high incidents (P2). The report includes system ID, incident timeline, impact assessment, and remediation steps.

**Q: How does this connect to FedRAMP ConMon?**

A: Every security incident on a FedRAMP-authorized system must be documented and reported to the 3PAO and CSP. Significant incidents may require a deviation request or updated POA&M. Our ConMon monthly report includes an incident summary. The ATO could be impacted if IR controls are found deficient during assessment.

---

*Prepared by: Omowumi Adisa, ISSO, NexusBridge Technologies*
*System: NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD*
*[GitHub Portfolio](https://github.com/Wunmi290)*
