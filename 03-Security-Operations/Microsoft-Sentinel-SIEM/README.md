# 📊 Microsoft Sentinel SIEM — Analytics Rules & SOAR Playbooks

**Category:** Security Operations | SIEM
**Environment:** Microsoft Sentinel (Azure Monitor)
**Outcome:** MTTR reduced by **35%** through automated detection and response
**Status:** ✅ Deployed in SOC environment

---

## 🎯 Objective

Engineer and tune Microsoft Sentinel analytics rules and SOAR (Security Orchestration, Automation and Response) playbooks to automate detection and response to recurring threat patterns, reducing analyst workload and mean time to respond.

---

## 🏗️ Architecture

```
Data Sources ──────────────────► Microsoft Sentinel Workspace
│                                         │
├── Microsoft Entra ID (Sign-in logs)     ├── Analytics Rules (KQL)
├── Microsoft 365 (Audit logs)            │     ├── Brute force detection
├── Azure Activity logs                   │     ├── Impossible travel
├── Microsoft Defender for Cloud          │     ├── Privileged role activation
├── On-premises AD (via AMA agent)        │     └── Legacy auth attempt
└── Network flow logs                     │
                                          ├── Incidents ──► SOAR Playbooks
                                          │     ├── Auto-block risky user
                                          │     ├── Auto-notify SOC via Teams
                                          │     └── Auto-revoke active sessions
                                          │
                                          └── Dashboards & Workbooks
```

---

## 🔍 Analytics Rules Created

### Rule 1: Brute Force Login Detection
**Logic:** 10+ failed sign-ins from same IP within 10 minutes, followed by success
**Severity:** High
**Response:** Trigger SOAR playbook — block IP, alert SOC, force MFA re-prompt

```kql
let threshold = 10;
let timeWindow = 10m;
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType != "0"  // Failed sign-ins
| summarize FailedAttempts = count(), FirstAttempt = min(TimeGenerated), 
            LastAttempt = max(TimeGenerated) by IPAddress, UserPrincipalName
| where FailedAttempts >= threshold
| join kind=inner (
    SigninLogs
    | where ResultType == "0"  // Successful sign-in
    | project SuccessTime = TimeGenerated, IPAddress, UserPrincipalName
) on IPAddress, UserPrincipalName
| where SuccessTime > LastAttempt
| project UserPrincipalName, IPAddress, FailedAttempts, FirstAttempt, SuccessTime
```

### Rule 2: Impossible Travel Detection
**Logic:** Successful sign-in from two geographically distant locations within 1 hour
**Severity:** Medium
**Response:** Alert SOC, flag account for review, require MFA re-verification

### Rule 3: Privileged Role Activation Outside Business Hours
**Logic:** PIM role activation by non-emergency account between 22:00–06:00 local time
**Severity:** Medium
**Response:** Alert Security Admin, log to audit trail

### Rule 4: Legacy Authentication Attempt
**Logic:** Any sign-in attempt using legacy protocols (SMTP, POP3, IMAP, NTLM)
**Severity:** Low (informational — should be zero after CA policy enforcement)
**Response:** Log and alert if volume exceeds 5/day (may indicate policy bypass)

---

## ⚡ SOAR Playbooks

### Playbook 1: Auto-Block Risky User
**Trigger:** High-severity identity alert (brute force success, credential spray)
**Actions:**
1. Revoke all active sessions for affected user (Entra ID)
2. Set account risk level to "High" in Identity Protection
3. Post alert to SOC Teams channel with user details and alert context
4. Create Sentinel incident with enriched entity data
5. Assign incident to on-call analyst

### Playbook 2: Phishing Email Auto-Response
**Trigger:** Defender for Office 365 high-confidence phishing detection
**Actions:**
1. Soft-delete email from all recipient mailboxes
2. Block sender domain in Exchange Online
3. Check if any user clicked links (URL detonation)
4. Alert SOC if clicks detected — escalate to Incident Response

---

## 📉 Outcomes

| Metric | Before Automation | After Automation |
|--------|------------------|--------------------|
| Mean Time to Respond (MTTR) | ~85 minutes | ~55 minutes (-35%) |
| Analyst triage time per alert | ~20 minutes | ~5 minutes (auto-enriched) |
| False positive rate | ~40% | ~18% (tuned rules) |
| Recurring alerts manually handled | ~25/week | ~4/week (automated) |

---

## 📋 Coming Soon
- `analytics-rules-export.json` — Sentinel rule templates (ARM format)
- `playbook-block-user.json` — Logic App playbook definition
- `kql-queries-library.md` — Reusable KQL detection queries

---

*Last updated: June 2026 | Platform: Microsoft Sentinel | MTTR improvement: 35%*
