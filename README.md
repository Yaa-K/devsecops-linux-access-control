# Linux User & Access Control – DevSecOps Practical Assessment

**Assignment 1 | ParoCyber DevSecOps Bootcamp**  
**Reflection Post:** [DEV.to – I Thought I Knew Linux. This Lab Proved Me Wrong.](https://dev.to/yaak/i-thought-i-knew-linux-this-lab-proved-me-wrong-2ljp) 

---

## Overview

This repository documents the hands-on completion of Assignment 1 of the ParoCyber DevSecOps Bootcamp. The assignment simulates real workplace scenarios involving Linux user management, access control, password state auditing, and department-based privilege enforcement.

All tasks were performed entirely in the Linux terminal, captured with screenshots, and documented with written explanations covering what was done, why it was done, observations made, and security implications.

---

## Repository Structure

```
├── README.md                        ← You are here
├── commands-reference.md            ← All commands used, organised by scenario
├── screenshots-guide.md             ← Index of every screenshot and what it proves
│
├── scenario-1-password-audit/
│   ├── README.md                    ← Full write-up for Scenario 1
│
└── scenario-2-user-access-config/
    ├── README.md                    ← Full write-up for Scenario 2
```

---

## Scenarios Summary

### Scenario 1 – Password State Investigation & Remediation

A simulated security audit where the task was to discover where Linux stores password state, identify users without passwords, and remediate the vulnerability — without being told where to look.

| Part | Task | Key Finding |
|------|------|-------------|
| A | Create user with no password | `useradd` without `-p` creates a passwordless account |
| B | Identify passwordless accounts | `/etc/shadow` second field shows `!` for no password |
| B | Non-privileged access test | Normal users cannot read `/etc/shadow` — Permission denied |
| C | Remediate and confirm | `passwd` changes `!` to a `$6$` SHA-512 hash |

### Scenario 2 – DevSecOps User & Department Access Configuration

A full onboarding simulation for 8 staff members across 4 department groups, plus a CI/CD service account — enforcing least privilege, department-based access, and audit-safe offboarding.

| Task | Description |
|------|-------------|
| 1 | Created all users using `useradd` and `adduser` |
| 2 | Created department groups and assigned users |
| 3 | Identified and remediated a passwordless account |
| 4 | Configured `ci_runner` with a non-login shell |
| 5 | Granted sudo to `alex` only — all others restricted |
| 6 | Removed `yaa` with `userdel` — home directory preserved |
| 7 | Verified group membership, group data storage, and privilege boundaries |

---

## Key Concepts Demonstrated

- Linux password state storage (`/etc/shadow` vs `/etc/passwd`)
- Department-based group access control
- Principle of least privilege via selective sudo assignment
- Service account security with non-login shells
- Forensic-safe user offboarding
- Audit traceability through system file inspection

---

## Tools & Commands Used

See [`commands-reference.md`](./commands-reference.md) for the full list.

Core utilities used: `useradd`, `adduser`, `userdel`, `usermod`, `groupadd`, `passwd`, `grep`, `getent`, `groups`, `sudo`

---

## Reflection

Read the full reflection post on DEV.to: [I Thought I Knew Linux. This Lab Proved Me Wrong.](https://dev.to/yaak/i-thought-i-knew-linux-this-lab-proved-me-wrong-2ljp)


