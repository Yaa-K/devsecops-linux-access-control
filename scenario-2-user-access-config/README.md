# Scenario 2 – DevSecOps User & Department Access Configuration

## Context

As a DevSecOps Engineer, onboard new staff onto a Linux server used for application development, security monitoring, QA, and CI/CD operations. Company policy enforces department-based access control, least privilege, and audit traceability.

---

## Staff & Department Assignments

| User | Department Group | Role |
|------|-----------------|------|
| lateef | dev_team | App Developer |
| arthur | dev_team | App Developer |
| purity | sec_team | Security & SOC |
| habiba | sec_team | Security & SOC |
| paula | qa_team | Quality Assurance |
| yaa | qa_team | Quality Assurance |
| alex | ops_team | Operations & Infra |
| aminat | ops_team | Operations & Infra |
| ci_runner | — | CI/CD Service Account |

---

## Task 1 – Create All Users

### Commands

```bash
# useradd — low-level, non-interactive
sudo useradd -m lateef
sudo useradd -m purity
sudo useradd -m arthur
sudo useradd -m paula
sudo useradd -m yaa
sudo useradd -m alex
sudo useradd -m habiba

# adduser — high-level, interactive
sudo adduser aminat

# Service account
sudo useradd -r -s /usr/sbin/nologin ci_runner
```

### Screenshots
- `2-1_useradd_users.png`
  <img width="1366" height="720" alt="Screenshot 2-1 useradd commands for lateef, purity, arthur, paula, yaa, alex, habiba" src="https://github.com/user-attachments/assets/1e137989-6797-4c4a-991e-1b934436de6e" />

- `2-2_adduser_aminat.png`
  <img width="1366" height="720" alt="Screenshot 2-2 adduser interactive creation for aminat" src="https://github.com/user-attachments/assets/c598e783-1bf3-4fc1-b94e-a279e80237f2" />

- `2-3_ci_runner_nologin.png`
  <img width="1366" height="720" alt="Screenshot 2-3 ci_runner service account creation with nologin shell" src="https://github.com/user-attachments/assets/389c7066-923a-4b20-9d7e-edbe5ba785f3" />


### Explanation

Both `useradd` and `adduser` create user accounts but serve different purposes. `useradd` is low-level and scriptable — ideal for automation pipelines. `adduser` is interactive and guides through setup including password creation. The `-m` flag ensures each user gets a home directory.

**Security Implication:** Individual accounts enforce accountability. Shared accounts make audit tracing impossible and violate least privilege principles.

---

## Task 2 – Create Groups & Assign Users

### Commands

```bash
sudo groupadd dev_team
sudo groupadd sec_team
sudo groupadd qa_team
sudo groupadd ops_team

sudo usermod -aG dev_team lateef
sudo usermod -aG dev_team arthur
sudo usermod -aG sec_team purity
sudo usermod -aG sec_team habiba
sudo usermod -aG qa_team paula
sudo usermod -aG qa_team yaa
sudo usermod -aG ops_team alex
sudo usermod -aG ops_team aminat

getent group dev_team
getent group sec_team
getent group qa_team
getent group ops_team
```

### Screenshots
- `2-4_groupadd_departments.png`
<img width="1366" height="720" alt="Screenshot 2-4 groupadd commands creating all four department groups" src="https://github.com/user-attachments/assets/35b6f3d5-cd65-470b-aa4b-d3a16471a666" />

`2-5_usermod_assignments.png`
  <img width="1366" height="720" alt="Screenshot 2-5 usermod -aG assignments for all users" src="https://github.com/user-attachments/assets/8cf29bd8-2d41-463f-8068-30f983ba7871" />

- `2-6_getent_group_verification.png`
  <img width="1366" height="720" alt="Screenshot 2-6 getent group output confirming all group memberships" src="https://github.com/user-attachments/assets/ff122f0c-aeaa-4bf3-8612-6beda0587bef" />


### Explanation

Linux records group membership in `/etc/group`. The `usermod -aG` command appends a supplementary group without removing existing ones (`-a` = append, `-G` = supplementary group). `getent group` retrieves this from the system database, confirming it is stored and recognised by the OS.

**Security Implication:** Group-based access control allows files and directories to be scoped to specific teams. A `sec_team` member cannot access `dev_team` resources by default — boundaries are enforced at the kernel level, not by application logic.

---

## Task 3 – Passwordless User: Identify & Remediate

### Commands

```bash
sudo grep "lateef" /etc/shadow   # BEFORE
sudo passwd lateef
sudo grep "lateef" /etc/shadow   # AFTER
```

### Screenshots
- `2-7_before_no_password.png`
  <img width="1366" height="720" alt="Screenshot 2-7 BEFORE — lateef etcshadow entry shows ! (no password)" src="https://github.com/user-attachments/assets/1d12dce6-ec03-4dfc-a553-7ad73df7b0ae" />

- `2-8_passwd_command.png`
<img width="1366" height="720" alt="Screenshot 2-8 passwd command setting lateef&#39;s password" src="https://github.com/user-attachments/assets/0f7c787b-7fca-4cf9-8ebc-cd7cb29f11f6" />

- `2-9_after_password_set.png`
  <img width="1366" height="720" alt="Screenshot 2-9 AFTER — lateef etcshadow shows hashed password" src="https://github.com/user-attachments/assets/aeea8f08-c527-4359-ae94-a0038f78a752" />


### Explanation

`lateef` was created without a password. The `/etc/shadow` entry showed `!` before remediation and `$6$...` after — confirming the fix.

**Security Implication:** Accounts reaching an active state without passwords indicate a broken provisioning process. In production, a forced password reset on first login should always accompany remediation.

---

## Task 4 – Configure ci_runner With Non-Login Shell

### Commands

```bash
grep "ci_runner" /etc/passwd
su - ci_runner
```

### Screenshots
- `2-10_ci_runner_passwd_entry.png`
  <img width="1366" height="720" alt="Screenshot 2-10 etcpasswd entry for ci_runner showing usrsbinnologin" src="https://github.com/user-attachments/assets/277f648f-20b4-43b5-a83b-df4a62c42605" />

- `2-11_ci_runner_login_denied.png`
  <img width="1366" height="720" alt="Screenshot 2-11 Attempted login to ci_runner — access denied" src="https://github.com/user-attachments/assets/67942a77-0350-4fbf-ab48-0423687ae656" />


### Explanation

`ci_runner` was created with `/usr/sbin/nologin` as its shell. The `/etc/passwd` entry confirms this. Attempting `su - ci_runner` returns: *"This account is currently not available"* — interactive login is blocked entirely.

**Security Implication:** Service accounts are high-value targets. They often carry elevated permissions for automated tasks. A non-login shell eliminates the risk of interactive session exploitation — even if credentials are compromised.

---

## Task 5 – Grant Sudo Access to One ops_team User

### Commands

```bash
sudo usermod -aG sudo alex
sudo -l -U alex      # Confirm alex has sudo
sudo -l -U aminat    # Confirm aminat does NOT
```

### Screenshots
- `2-12_usermod_sudo_alex.png`
  <img width="1366" height="720" alt="Screenshot 2-12 usermod adding alex to sudo group" src="https://github.com/user-attachments/assets/3c044d93-34a5-4243-9c35-a4a6e42da7ae" />

- `2-13_alex_sudo_confirmed.png`
  <img width="1366" height="720" alt="Screenshot 2-13 sudo -l -U alex confirming elevated privileges" src="https://github.com/user-attachments/assets/fe9f30a8-9ac1-4b1f-bfb4-9cea32dc3164" />

- `2-14_aminat_no_sudo.png`
<img width="1366" height="720" alt="Screenshot 2-14 sudo -l -U aminat showing NO sudo access" src="https://github.com/user-attachments/assets/fac91304-cae4-4460-a5bf-902f3a9c68bc" />

### Explanation

Only `alex` from `ops_team` received sudo access. `aminat`, also in `ops_team`, was deliberately excluded to enforce least privilege. `sudo -l -U aminat` confirmed: *"User aminat is not allowed to run sudo on this host."*

**Security Implication:** Sudo access must be granted to named individuals, not broad groups. Every sudo action is logged in `/var/log/auth.log`, providing the audit trail required for compliance and incident response in DevSecOps environments.

---

## Task 6 – Remove User, Preserve Home Directory

### Commands

```bash
sudo userdel yaa          # No -r flag — preserves home directory
grep "yaa" /etc/passwd    # Confirm account removed
ls -la /home/yaa          # Confirm directory preserved
```

### Screenshots
- `2-15_userdel_yaa.png`
  <img width="1366" height="720" alt="Screenshot 2-15 userdel yaa command execution" src="https://github.com/user-attachments/assets/21de51f1-4b88-4785-afd9-ada905ea9043" />

- `2-16_yaa_removed_passwd.png`
  <img width="1366" height="720" alt="Screenshot 2-16 grep etcpasswd confirming yaa is removed" src="https://github.com/user-attachments/assets/ec6fee24-658a-40a3-ac45-33b2ee72ab7b" />

- `2-17_home_directory_preserved.png`
<img width="1366" height="720" alt="Screenshot 2-17 ls homeyaa confirming directory is preserved" src="https://github.com/user-attachments/assets/7b613301-5866-454f-ad76-be3724cfd1ea" />

### Explanation

`userdel` without `-r` removes the user account but leaves `/home/yaa` intact. `grep "yaa" /etc/passwd` returned no output — account gone. `ls /home/yaa` still listed the directory contents — data preserved.

**Security Implication:** A departed user's home directory may contain scripts, logs, or configuration files relevant to a security investigation. Deleting it immediately could destroy forensic evidence and violate data retention policies. Preservation is standard practice in DevSecOps offboarding.

---

## Task 7 – Verification

### Commands

```bash
for user in lateef purity arthur paula yaa alex habiba aminat; do
  echo "--- $user ---"
  groups $user
done

cat /etc/group | grep -E "dev_team|sec_team|qa_team|ops_team"

sudo -l -U alex
sudo -l -U lateef
```

### Explanation

This task closes the accountability loop — proving configurations are not just applied but verifiably enforced at the system level. Every user appeared in the correct group. `/etc/group` confirmed all four department groups. `alex` had sudo; `lateef` did not.

**Security Implication:** In production, this verification step would be automated as part of a post-onboarding compliance check, feeding results into a SIEM or audit log for continuous monitoring.

---

## Conceptual Analysis

**How does Linux record and track group membership?**
Linux stores group data in `/etc/group`, where each line follows the format `groupname:password:GID:members`. When a user is assigned to a supplementary group via `usermod -aG`, their username is appended to that group's entry. The kernel reads this at login to build the user's full group list, which determines what shared resources they can access.

**How does department-based grouping enforce access control?**
File and directory permissions in Linux operate across three levels: owner, group, and others. By scoping files to a specific group, only members of that group can access them. A `sec_team` member cannot access `dev_team` directories — the boundary is enforced by the kernel, not by application logic.

**Why must service accounts use non-login shells?**
Service accounts are designed for automated processes, not human interaction. A non-login shell like `/usr/sbin/nologin` ensures no interactive session can ever be opened — eliminating an entire attack vector even if credentials are leaked.

**Why are home directories retained after user deletion?**
A user's home directory may contain forensic evidence needed for security investigations or compliance audits. Deleting it immediately could destroy that evidence and violate data retention policies. Retention is often a legal requirement in regulated environments.

**How is privilege escalation controlled in DevSecOps environments?**
Through the principle of least privilege — users receive only the minimum access required for their role. Sudo is granted selectively to named individuals, every action is logged, and access reviews are part of regular compliance checks.
