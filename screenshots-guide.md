# Screenshots Guide

**Assignment 1 – Linux User & Access Control**  
This file explains what every screenshot shows and what evidence it provides.

---

## Scenario 1 – Password State Investigation & Remediation

### Part A – User Creation

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `1A-1_useradd_command.png` | `sudo useradd audit_test` executed in terminal | User was created without a password flag |
| `1A-2_user_exists_passwd.png` | `grep "audit_test" /etc/passwd` output | Account exists with valid UID, GID, home dir, and shell |

---

### Part B – Password State Discovery

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `1B-1_shadow_inspection.png` | `sudo cat /etc/shadow` full output | `/etc/shadow` is where Linux stores password state |
| `1B-2_filtered_grep_no_password.png` | Filtered grep showing `audit_test:!:...` | The `!` confirms no password is set on the account |
| `1B-3_normal_user_permission_denied.png` | `cat /etc/shadow` run as a normal user | Non-privileged users cannot access `/etc/shadow` — Permission denied |

---

### Part C – Remediation

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `1C-1_before_no_password.png` | `/etc/shadow` entry for `audit_test` before remediation | Password field shows `!` — no password |
| `1C-2_passwd_command.png` | `sudo passwd audit_test` with password prompt | Password is being set interactively |
| `1C-3_after_password_set.png` | `/etc/shadow` entry for `audit_test` after remediation | Password field now shows `$6$...` hash — remediated |

---

## Scenario 2 – DevSecOps User & Department Access Configuration

### Task 1 – User Creation

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `2-1_useradd_users.png` | `useradd -m` commands for 7 users | Users created with home directories using low-level tool |
| `2-2_adduser_aminat.png` | `adduser aminat` interactive session | aminat created using high-level interactive tool |
| `2-3_ci_runner_nologin.png` | `useradd -r -s /usr/sbin/nologin ci_runner` | Service account created with non-login shell |

---

### Task 2 – Group Creation & Assignment

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `2-4_groupadd_departments.png` | `groupadd` for all 4 department groups | All department groups created successfully |
| `2-5_usermod_assignments.png` | `usermod -aG` commands for all users | All users assigned to correct department groups |
| `2-6_getent_group_verification.png` | `getent group` output for all 4 groups | Group membership is stored and retrievable from the system |

---

### Task 3 – Password Remediation

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `2-7_before_no_password.png` | `grep "lateef" /etc/shadow` — before | lateef has no password (`!` in password field) |
| `2-8_passwd_command.png` | `sudo passwd lateef` with prompt | Password being set for lateef |
| `2-9_after_password_set.png` | `grep "lateef" /etc/shadow` — after | lateef now has a `$6$` hash — remediated |

---

### Task 4 – ci_runner Non-Login Shell

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `2-10_ci_runner_passwd_entry.png` | `grep "ci_runner" /etc/passwd` | Last field shows `/usr/sbin/nologin` as shell |
| `2-11_ci_runner_login_denied.png` | `su - ci_runner` attempt | Returns: "This account is currently not available" |

---

### Task 5 – Sudo Access

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `2-12_usermod_sudo_alex.png` | `sudo usermod -aG sudo alex` | alex added to sudo group |
| `2-13_alex_sudo_confirmed.png` | `sudo -l -U alex` output | alex has sudo privileges confirmed |
| `2-14_aminat_no_sudo.png` | `sudo -l -U aminat` output | aminat is NOT allowed to run sudo — least privilege enforced |

---

### Task 6 – User Removal

| File | What It Shows | Evidence Provided |
|------|--------------|-------------------|
| `2-15_userdel_yaa.png` | `sudo userdel yaa` (no `-r` flag) | User account deleted without removing home directory |
| `2-16_yaa_removed_passwd.png` | `grep "yaa" /etc/passwd` returns nothing | yaa no longer exists as a system user |
| `2-17_home_directory_preserved.png` | `ls -la /home/yaa` shows contents | Home directory intact — preserved for forensic/audit purposes |

---

## Screenshot Naming Convention

All screenshots follow this format:

```
[scenario]-[task]-[description].png

Examples:
1A-1_useradd_command.png       ← Scenario 1, Part A, screenshot 1
2-6_getent_group_verification  ← Scenario 2, Task 6
```

This makes it easy to match each screenshot to the relevant section of the write-up and submission document.
