# 🔐 Entra ID Hybrid Deployment — 200-User Environment

**Category:** Identity & Access Management | Zero Trust
**Environment:** Microsoft Azure + On-Premises Active Directory (Hybrid)
**Scale:** 200 users | 10+ SaaS integrations | 3 locations
**Status:** ✅ Deployed in production environment

---

## 🎯 Objective

Design and deploy a secure hybrid identity architecture using Microsoft Entra ID (Azure AD) to unify cloud and on-premises identity management, enforce Zero Trust access controls, and eliminate identity gaps across the organisation.

---

## 🏗️ Architecture Overview

```
On-Premises AD ──── Azure AD Connect ──── Microsoft Entra ID (Cloud)
                                                    │
                    ┌───────────────────────────────┼──────────────────────────────┐
                    │                               │                              │
            Conditional Access              MFA Enforcement               SSO (10+ Apps)
            Policies (6 rules)              (All users)                  (SAML/OIDC)
                    │                               │                              │
            Identity-Based               Azure AD App Proxy            SaaS Platforms:
            Access Controls              (Internal apps)               - Microsoft 365
                                                                        - Slack, Zoom
                                                                        - GitHub, Jira
                                                                        - HR/Payroll SaaS
```

---

## ⚙️ What Was Deployed

### 1. Azure AD Connect (Hybrid Identity Sync)
- Installed and configured Azure AD Connect for password hash synchronisation
- Synced 200 on-premises AD accounts to Entra ID
- Configured filtered sync (OU-based) to exclude service accounts from cloud sync
- Verified sync health via Azure AD Connect Health dashboard

### 2. Conditional Access Policies (6 Policies)
| Policy | Trigger | Action |
|--------|---------|--------|
| Require MFA — All Users | Any sign-in outside trusted network | Block unless MFA passed |
| Block Legacy Authentication | Legacy auth protocols (SMTP, IMAP, POP3) | Block always |
| Require Compliant Device | Access to M365 apps | Block non-Intune enrolled devices |
| Location-Based Access | Sign-in from high-risk countries | Block |
| Admin Role Protection | Any privileged role sign-in | Require MFA + compliant device |
| Sign-in Risk Policy | Azure AD Identity Protection risk score ≥ Medium | Force password reset |

### 3. MFA Enforcement
- Enforced MFA via Conditional Access for 100% of users (not per-user MFA legacy method)
- Configured Microsoft Authenticator as primary MFA method
- Set up SSPR (Self-Service Password Reset) to reduce helpdesk load
- Excluded emergency break-glass accounts from MFA policy (separate monitoring alert configured)

### 4. SSO Integrations (10+ SaaS Platforms)
- Configured SAML 2.0 SSO for: Microsoft 365, Slack, Zoom, GitHub Enterprise, Jira, Confluence
- Configured OIDC SSO for: HR platform, project management tool, document signing platform
- Tested all integrations end-to-end with test users before production rollout
- Documented SP-initiated vs IdP-initiated flows per application

### 5. Azure AD Application Proxy
- Deployed Application Proxy connectors on two on-premises servers for redundancy
- Published 3 internal web applications via Application Proxy — eliminating VPN requirement for those apps
- Configured pre-authentication with Entra ID before access is granted to internal app

---

## 🔒 Security Outcomes

| Metric | Before | After |
|--------|--------|-------|
| MFA Coverage | ~20% (ad hoc) | 100% enforced |
| Legacy auth blocked | ❌ Not blocked | ✅ Fully blocked |
| SSO coverage | 0 apps | 10+ apps |
| Identity gaps (cloud vs on-prem) | Multiple ungoverned accounts | ✅ Unified, governed |
| Admin accounts with standing access | Multiple | 0 (moved to PIM — see PIM lab) |

---

## 🧠 Key Decisions & Reasoning

**Why password hash sync over pass-through auth?**
Password hash sync provides authentication resilience — users can still authenticate if on-premises AD goes down. For a 3-location organisation with limited IT resources, availability was prioritised over the marginal security benefit of PTA.

**Why Conditional Access MFA over per-user MFA?**
Per-user MFA is a legacy method that creates inconsistent enforcement. Conditional Access MFA is policy-driven, auditable, and integrates with risk signals — making it far more operationally sound.

**Why block legacy authentication?**
Legacy protocols don't support MFA. Any MFA deployment is undermined if legacy auth remains open — attackers simply target SMTP or IMAP endpoints. Blocking legacy auth was non-negotiable before declaring MFA coverage complete.

---

## ⚠️ Challenges & How I Resolved Them

**Challenge 1: Sync conflict — duplicate UPN attributes**
Several on-premises accounts had UPNs that conflicted with M365 licensed accounts. Azure AD Connect flagged these as sync errors.
*Resolution:* Used IdFix tool to identify and clean up UPN mismatches in on-premises AD before re-running sync. Documented clean-up process for future account provisioning.

**Challenge 2: MFA rollout causing helpdesk spike**
First wave of MFA enforcement caused a spike in helpdesk tickets from users who hadn't set up the Authenticator app.
*Resolution:* Created a 2-week phased rollout with a named location exclusion. Ran an organisation-wide Authenticator setup session before enforcement date. Helpdesk spike resolved within 3 days.

**Challenge 3: Application Proxy intermittent timeout on one internal app**
One internal application published via App Proxy was experiencing session timeouts within 10 minutes.
*Resolution:* Investigated and found the app's session timeout was shorter than the App Proxy token lifetime. Configured backend app to honour Azure AD session tokens and extended idle session timeout.

---

## 📋 Files in This Folder

| File | Description |
|------|-------------|
| README.md | This document — full architecture and implementation notes |
| conditional-access-policies.md | Detailed CA policy configurations and targeting rules *(coming soon)* |
| sso-integration-checklist.md | Step-by-step SSO setup checklist used during rollout *(coming soon)* |
| mfa-rollout-plan.md | Phased MFA rollout plan template *(coming soon)* |

---

## 🔗 Related Labs
- [Privileged Identity Management](../Privileged-Identity-Management/) — JIT access built on top of this Entra ID foundation
- [Zero Trust Network Segmentation](../Zero-Trust-Network-Segmentation/) — App Proxy extended as part of broader Zero Trust initiative
- [Microsoft Sentinel SIEM](../../03-Security-Operations/Microsoft-Sentinel-SIEM/) — Entra ID sign-in logs fed into Sentinel for threat detection

---

*Last updated: June 2026 | Environment: Production-adjacent | All sensitive data anonymised*
