# SX-03: Cryptography and PKI Controls Reference Guide

**Project Title:** Cryptography and PKI Controls Reference Guide for NexusBridge Grant Management System
**System Reference:** NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD
**Framework / Standard:** NIST SP 800-57 Part 1 Rev 5 | NIST SP 800-52 Rev 2 | CompTIA Security+ SY0-701 Domain 3
**Author:** Omowumi Adisa, ISSO, NexusBridge Technologies
**Date:** May 2024
**Status:** Complete

## Purpose

Cryptography is one of the most tested areas on the CompTIA Security+ SY0-701 exam and one of the most critical components of federal information system security. This document serves as a practical reference guide for all cryptographic controls deployed in NexusBridge GMS, mapping each control to the specific use case, algorithm, key length, and relevant NIST guidance.

## Section 1: Cryptography Inventory for NexusBridge GMS

| Control | Use Case | Algorithm | Key Length | NIST Reference | Implementation |
|---|---|---|---|---|---|
| Data at Rest (RDS) | PostgreSQL database encryption | AES-256-GCM | 256-bit | NIST SP 800-111, FIPS 140-3 | AWS KMS CMK + RDS storage encryption |
| Data at Rest (S3) | Backup and log storage encryption | AES-256 (SSE-KMS) | 256-bit | NIST SP 800-111 | AWS KMS CMK applied to all S3 buckets |
| Data at Rest (EBS) | EC2 instance root and data volumes | AES-256 (SSE-KMS) | 256-bit | NIST SP 800-111 | AWS KMS CMK; all volumes encrypted at creation |
| Data in Transit (TLS) | All HTTPS connections (ALB to clients, app to RDS) | TLS 1.3 preferred, TLS 1.2 minimum | N/A (cipher suite dependent) | NIST SP 800-52 Rev 2 | ALB TLS security policy: ELBSecurityPolicy-TLS13-1-2-2021-06 |
| TLS Cipher Suites | HTTPS encryption | ECDHE-ECDSA-AES256-GCM-SHA384, ECDHE-RSA-AES256-GCM-SHA384 | 256-bit | NIST SP 800-52 Rev 2 | Configured in ALB and application server |
| API Transaction Integrity | Treasury Disbursement API payload signing | HMAC-SHA256 | 256-bit (HMAC key) | NIST SP 800-107 Rev 1 | Application-layer signing; verified by Treasury endpoint |
| Key Management | All NexusBridge KMS keys | RSA-2048 (key wrapping), AES-256 (data keys) | 256-bit (data keys) | NIST SP 800-57 Part 1 | AWS KMS Customer Managed Keys (CMKs) |
| Key Rotation | Automatic rotation of KMS CMKs | Annual rotation | N/A | NIST SP 800-57 Part 1 | AWS KMS automatic rotation enabled |
| Code Signing | Application deployment artifacts | SHA-256 | 256-bit | NIST SP 800-131A Rev 2 | GitHub Actions pipeline signs container images |

## Section 2: Symmetric Encryption

**Definition:** Symmetric encryption uses the same key for both encryption and decryption.

**NexusBridge Primary Algorithm:** AES (Advanced Encryption Standard), specifically AES-256 in GCM (Galois/Counter Mode).

**Why AES-256-GCM:**
- AES is approved by NIST and FIPS for federal government use (FIPS 197).
- 256-bit key length provides resistance to brute force with current and foreseeable quantum computing capabilities.
- GCM mode provides both confidentiality (encryption) and integrity (authentication tag). This is an Authenticated Encryption with Associated Data (AEAD) mode.
- GCM is more efficient than CBC mode and does not require padding.

**Use Cases in NexusBridge GMS:** RDS database encryption at rest, S3 bucket object encryption, EBS volume encryption.

**NIST Reference:** NIST SP 800-38D (GCM mode), FIPS 197 (AES), NIST SP 800-111 (storage encryption)

## Section 3: Asymmetric Encryption and Key Exchange

**Definition:** Asymmetric encryption uses a public/private key pair.

| Use Case | Algorithm | Key Length | Notes |
|---|---|---|---|
| TLS session key establishment | ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) | P-256 or P-384 | Provides perfect forward secrecy; each session uses a new ephemeral key |
| TLS certificate (ALB) | RSA-2048 with SHA-256 signature | 2048-bit | AWS Certificate Manager (ACM) issues and manages the TLS certificate |
| Mutual TLS (Treasury API) | RSA-2048 client certificate | 2048-bit | NexusBridge presents client certificate to Treasury API; mutual authentication |

**Why Perfect Forward Secrecy (PFS) Matters:** ECDHE provides PFS because the server's private key is not used to encrypt session keys. Even if an attacker records encrypted traffic and later compromises the server's private key, they cannot decrypt historical sessions.

**NIST Reference:** NIST SP 800-56A Rev 3 (key establishment), NIST SP 800-186 (elliptic curve parameters)

## Section 4: Hashing

**Definition:** A cryptographic hash function takes an input of any length and produces a fixed-length output (digest).

| Algorithm | Output Length | NexusBridge Use Case | Status |
|---|---|---|---|
| SHA-256 | 256 bits | HMAC transaction signing (Treasury API); code signing; TLS certificate signatures | Approved (NIST SP 800-131A Rev 2) |
| SHA-384 | 384 bits | TLS cipher suite MAC (ECDHE-ECDSA-AES256-GCM-SHA384) | Approved |
| SHA-1 | 160 bits | Legacy certificate on monitoring-01 (Finding 5 from vulnerability scan) | Deprecated. Remediation in progress. |
| MD5 | 128 bits | Not in use | Not approved for cryptographic purposes |

**Why SHA-1 is Deprecated:** SHA-1 was formally deprecated by NIST in NIST SP 800-131A Rev 2 (2019). Theoretical collision attacks against SHA-1 became practical in 2017 (SHAttered attack). NexusBridge is migrating the monitoring server certificate to SHA-256 by December 31, 2024.

**HMAC Explained:** HMAC (Hash-based Message Authentication Code) combines a cryptographic key with a hash function. NexusBridge uses HMAC-SHA256 for Treasury API transaction signing, providing both integrity (the data was not modified) and authenticity (only the party with the shared secret could have generated the tag).

## Section 5: PKI and Certificate Management

| Certificate | Issued By | Type | Use | Expiration | Management |
|---|---|---|---|---|---|
| ALB TLS Certificate (nexusbridge-gms.doe.gov) | Amazon Certificate Manager (ACM) | Domain Validation (DV) | HTTPS termination at ALB | Auto-renewed by ACM | AWS ACM |
| Internal Code Signing Certificate | GitHub Actions (internal CA) | Code signing | Container image signing | 1 year | GitHub Secrets |
| Mutual TLS Client Certificate (Treasury API) | Treasury PKI | Client authentication | Authenticate NexusBridge to Treasury API | 2 years | AWS Secrets Manager |
| Monitoring Server Certificate (monitoring-01) | Self-signed | Self-signed | Internal only | 2 years | Manual (action item) |

**Certificate Lifecycle Management:**
- **Issuance:** AWS ACM automatically provisions TLS certificates for public-facing endpoints via DNS validation with Route 53.
- **Renewal:** ACM automatically renews certificates 60 days before expiration.
- **Revocation:** Managed via ACM (for ACM-managed certificates) or by contacting the issuing CA. Revocation status checked via OCSP.

## Section 6: Post-Quantum Cryptography Planning

NIST finalized the first post-quantum cryptography (PQC) standards in 2024 (FIPS 203, 204, 205). NexusBridge GMS is monitoring NIST PQC guidance. The current architecture uses algorithm-agnostic libraries (OpenSSL 3.x) that support PQC extensions. No immediate migration is required, but the ISSO will include PQC readiness in the 2025 annual risk assessment.

## Interview Defense Notes

**What is the difference between symmetric and asymmetric encryption?**
Symmetric uses the same key for encryption and decryption (faster, used for bulk data: AES). Asymmetric uses a public/private key pair (slower, used for key exchange and signatures: RSA, ECDSA). TLS uses asymmetric for key exchange and symmetric for the bulk data stream.

**What is perfect forward secrecy and why does NexusBridge require it?**
PFS means that compromising the server's private key in the future cannot decrypt previously recorded sessions. ECDHE provides PFS by generating a new ephemeral key for each session. All NexusBridge TLS cipher suites require ECDHE for PFS.

**What is HMAC and how is it different from a regular hash?**
A regular hash provides integrity (has the data changed) but not authenticity (who created it). HMAC adds a secret key to the hash, so only parties with the key can produce a valid HMAC. NexusBridge uses HMAC-SHA256 for the Treasury API to ensure payment instructions came from NexusBridge, not an impersonator.

**Why is AES-256 preferred over AES-128 for federal systems?**
NIST SP 800-57 recommends AES-256 for data that needs to remain confidential beyond 2030. For federal grant data with long retention requirements, AES-256 provides a larger security margin against future advances in computing power, including potential quantum threats.

---
*Prepared by: Omowumi Adisa, ISSO, NexusBridge Technologies*
*System: NexusBridge Grant Management System (GMS) | DOE-NB-GMS-2024-MOD*
*[GitHub Portfolio](https://github.com/Wunmi290)*
