# SX-04b: IAM Access Review and Provisioning Workflow

**System Reference:** NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD
**Security Domain:** Security Operations
**Framework Alignment:** NIST SP 800-53 AC/IA Families, FedRAMP Moderate Baseline
**Classification:** Controlled Unclassified Information (CUI)

---

## Overview

This document describes the Identity and Access Management (IAM) architecture, account provisioning workflow, and quarterly access review process for the NexusBridge Grant Management System (GMS). It supports FedRAMP AC and IA control families and demonstrates the ISSO's ability to enforce least privilege, manage privileged access, and maintain audit-ready access logs.

---

## Federated Identity Architecture

| Component | Technology | Purpose |
|---|---|---|
| Identity Provider (IdP) | AWS IAM Identity Center (SSO) | Centralized authentication for GMS users |
| Directory Service | DOE Active Directory (AD) | Authoritative source for user identity |
| MFA Provider | Duo Security (FIPS-compliant) | Step-up authentication for all privileged roles |
| Federation Protocol | SAML 2.0 | DOE AD to AWS IAM Identity Center |
| Access Control Model | Role-Based Access Control (RBAC) | Enforce least privilege by job function |
| Secrets Management | AWS Secrets Manager | Rotate service account credentials automatically |

---

## Role Definitions (RBAC Matrix)

| Role | Job Function | GMS Access Level | AWS Permissions | MFA Required |
|---|---|---|---|---|
| ISSO | Omowumi Adisa | Read + audit logs | SecurityAudit policy | Yes |
| System Administrator | Jerome K. Okafor (Cloud Ops) | Full admin (break-glass) | AdministratorAccess (time-limited) | Yes (Duo) |
| Grant Reviewer | DOE Program Officer | Read grant records | gms-grant-reviewer-policy | Yes |
| Data Entry Analyst | DOE Grant Coordinator | Create/edit grant records | gms-data-entry-policy | Yes |
| Read-Only Auditor | 3PAO Sandra L. Pryce | Read-only all data | gms-auditor-readonly-policy | Yes |
| Application Service Account | NexusBridge GMS API | API calls only | gms-svc-account-policy (least priv) | N/A (key rotation) |

---

## Account Provisioning Workflow

**Step 1: Access Request**
- User or manager submits access request via ServiceNow ITSM
- Request includes: job title, business justification, required role, supervisor approval
- Request routed to ISSO (Omowumi Adisa) for review

**Step 2: ISSO Review and Approval**
- ISSO verifies business justification aligns with least privilege principle
- Confirm user has completed required security awareness training (NexusBridge GMS-SAT-001)
- Verify DOE employee or contractor status via HR system
- Approve or deny in ServiceNow within 2 business days

**Step 3: Account Creation**
- Cloud Ops Lead (Jerome K. Okafor) creates account in AWS IAM Identity Center
- Assigns appropriate permission set (RBAC role) per ISSO approval
- MFA enrollment email sent to user (Duo Security)
- Account remains disabled until MFA enrollment confirmed

**Step 4: MFA Enrollment and Activation**
- User enrolls Duo MFA device within 24 hours
- Cloud Ops activates account and assigns GMS portal access
- Welcome email sent with acceptable use policy acknowledgment link

**Step 5: Onboarding Documentation**
- ServiceNow ticket closed with provisioning confirmation
- Access record logged in NexusBridge IAM Register (IAM-REG-2024)
- System Owner (Dr. Marcus T. Webb) notified for privileged roles

---

## Quarterly Access Review Process

**Schedule:** January, April, July, October (aligned to FedRAMP ConMon cycle)

**Process:**

1. ISSO generates IAM user/role report from AWS IAM Identity Center
2. ISSO exports report into Access Review Worksheet (ARW-Q[X]-2024)
3. Each role owner (manager) receives worksheet to certify or revoke access
4. Managers return certified worksheet within 5 business days
5. ISSO processes revocations: submits deprovisioning tickets in ServiceNow
6. Cloud Ops removes access within 2 business days of ticket submission
7. ISSO documents review results in POA&M if anomalies found
8. System Owner (Dr. Marcus T. Webb) signs off on quarterly review summary
9. Review artifacts archived for FedRAMP evidence package

**Review Metrics Tracked:**

| Metric | Target | Q1 2024 Actual |
|---|---|---|
| Accounts reviewed | 100% | 100% (47 accounts) |
| Manager certification rate | 100% within 5 days | 96% (2 late) |
| Revocations processed | Within 2 business days | 100% (8 revocations) |
| Orphaned accounts found | 0 | 1 (terminated contractor) |
| Excessive privilege findings | 0 | 2 (downgraded to read-only) |

---

## Deprovisioning Workflow

**Trigger Events:** Termination, role change, leave of absence, contract expiration, access review revocation

| Step | Action | Owner | Timeline |
|---|---|---|---|
| 1 | HR notifies Cloud Ops and ISSO of termination/role change | DOE HR | Day of event |
| 2 | ISSO suspends account in AWS IAM Identity Center | Omowumi Adisa | Within 1 hour (termination) |
| 3 | Revoke all active sessions and API keys | Jerome K. Okafor | Within 1 hour |
| 4 | Remove from all distribution lists and shared mailboxes | Cloud Ops | Within 24 hours |
| 5 | Transfer data ownership to manager in DOE SharePoint | Cloud Ops | Within 48 hours |
| 6 | Archive IAM logs for the departed user (90-day retention) | ISSO | Within 72 hours |
| 7 | Close ServiceNow ticket and update IAM Register | ISSO | Within 72 hours |

---

## Privileged Access Management (PAM)

**Privileged roles** (System Administrator, Database Admin, Cloud Ops Lead) require:

- Separate privileged account (e.g., padmin-jokafor) distinct from standard user account
- Just-In-Time (JIT) access provisioning via AWS IAM Identity Center session policies
- All privileged sessions logged to AWS CloudTrail with immutable log storage (S3 with Object Lock)
- Session duration limited to 4 hours maximum (auto-expiration)
- Dual approval required for break-glass access (ISSO + System Owner)
- Monthly privileged account review by ISSO

---

## NIST SP 800-53 Control Mapping

| Control | Title | Implementation |
|---|---|---|
| AC-2 | Account Management | Provisioning/deprovisioning workflow above |
| AC-3 | Access Enforcement | RBAC permission sets in AWS IAM Identity Center |
| AC-5 | Separation of Duties | Separate standard and privileged accounts |
| AC-6 | Least Privilege | Scoped permission policies per job function |
| AC-17 | Remote Access | MFA required for all remote access (Duo) |
| IA-2 | Identification and Authentication | AWS IAM Identity Center + Duo MFA |
| IA-4 | Identifier Management | Unique user IDs, no shared accounts |
| IA-5 | Authenticator Management | MFA enrollment required before activation |
| AC-2(1) | Automated System Account Management | ServiceNow workflow automation |

---

## Interview Defense Notes

**Q: What is least privilege and how do you enforce it?**

A: Least privilege means users only have the minimum access required to perform their job functions. I enforce it through our RBAC model: each role has a scoped permission policy that limits actions to only what is needed. For example, a Grant Reviewer can read grant records but cannot modify or delete them. During provisioning, I review every access request against the user's job title and business justification before approving.

**Q: How do you handle a terminated employee's access?**

A: Termination triggers an immediate response. Within 1 hour, I suspend the account in AWS IAM Identity Center and Jerome (Cloud Ops) revokes all active sessions. We follow a documented deprovisioning checklist in ServiceNow. Data ownership is transferred to the manager within 48 hours. IAM logs are archived for 90 days for forensic purposes. This maps directly to NIST AC-2.

**Q: What is a quarterly access review and why is it important?**

A: A quarterly access review (also called access recertification) is when I generate an IAM report and send it to each role owner to confirm all access is still appropriate. Managers certify or revoke access within 5 business days. This catches orphaned accounts (terminated employees not yet deprovisioned) and access creep (users accumulating permissions over time). For FedRAMP, access review artifacts are required evidence in the ConMon package.

**Q: What is the difference between authentication and authorization?**

A: Authentication verifies who you are (identity): username, password, MFA. Authorization determines what you can do (access): which resources and operations. In our system, AWS IAM Identity Center handles authentication via SAML federation with DOE AD plus Duo MFA. Authorization is enforced by IAM permission sets (RBAC policies). A user must first authenticate successfully before the authorization policy is evaluated.

---

*Prepared by: Omowumi Adisa, ISSO, NexusBridge Technologies*
*System: NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD*
*[GitHub Portfolio](https://github.com/Wunmi290)*
