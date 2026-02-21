# Scenario 1 – Password State Investigation & Remediation

## Context

During a routine security audit, the Security Director asks:
> *"Show me how you identify users on this system who do not have passwords set, and then demonstrate how you would remediate one."*

The location of the relevant system file was not provided. Discovery was part of the task.

---

## Part A – Create a User With No Password

### Commands

```bash
sudo useradd audit_test
grep "audit_test" /etc/passwd
```

### Screenshots
- `1A-1_useradd_command.png`
  <img width="1366" height="720" alt="Screenshot 1A-1 useradd command execution" src="https://github.com/user-attachments/assets/18cd8778-47d7-4673-b13b-93d2f1951d8f" />

- `1A-2_user_exists_passwd.png`
<img width="1366" height="720" alt="Screenshot 1A-2 grep etcpasswd confirming user exists" src="https://github.com/user-attachments/assets/5a43573f-be84-4760-b2b1-c73afd4e9d19" />

### Explanation

`useradd` without a password flag creates an account with no password set. The account immediately appears in `/etc/passwd` with a valid UID, GID, home directory, and default shell — confirming the user exists on the system.

**Security Implication:** A user account with no password is a misconfiguration. Depending on PAM configuration, it could allow unauthorized access. Identifying such accounts is a fundamental step in any access control audit.

---

## Part B – Identify Users Without Passwords

### Discovery

Password state in Linux is not stored in `/etc/passwd` — that file is world-readable and only holds the `x` placeholder. The actual password data lives in `/etc/shadow`, which is readable only by root.

### Commands

```bash
sudo cat /etc/shadow
sudo grep "audit_test" /etc/shadow
sudo grep -E "^[^:]+:(!|\*|::)" /etc/shadow

# Non-privileged access test
cat /etc/shadow
```

### Screenshots
- `1B-1_shadow_inspection.png`
  <img width="1366" height="720" alt="Screenshot 1B-1 etcshadow inspection showing password state" src="https://github.com/user-attachments/assets/77a68098-dccb-4ef0-876e-2c607ba33440" />

- `1B-2_filtered_grep_no_password.png`
  <img width="1366" height="720" alt="Filtered grep output highlighting accounts with no password" src="https://github.com/user-attachments/assets/40a3ea0e-6a05-4931-a3b7-f9104354e25e" />

- `1B-3_normal_user_permission_denied.png`
  <img width="1366" height="720" alt="Screenshot 1B-3 Permission denied — normal user cannot read etcshadow" src="https://github.com/user-attachments/assets/3621f0cd-1ff4-4df4-be66-a3d4254a6056" />


### Explanation

The second field in `/etc/shadow` holds the password hash. A `!` means no password is set. An empty field means anyone can log in without a password. A `$6$` prefix means a valid SHA-512 hash exists.

For `audit_test`, the field showed `!` — confirming no password was set.

The non-privileged access test returned `Permission denied`, confirming that `/etc/shadow` is intentionally restricted to prevent credential harvesting.

**Security Implication:** The separation between the world-readable `/etc/passwd` and the root-only `/etc/shadow` is a deliberate, foundational Linux security design. It ensures that password hashes are never exposed to unprivileged processes or users.

### /etc/shadow Field Reference

| Symbol | Meaning | Risk |
|--------|---------|------|
| `!` | Locked / no password | High |
| `*` | Login disabled | Low — intentional |
| `$6$...` | Valid SHA-512 hash | Normal |
| *(empty)* | No password — open login | Critical |

---

## Part C – Remediation

### Commands

```bash
# BEFORE
sudo grep "audit_test" /etc/shadow

# Set password
sudo passwd audit_test

# AFTER
sudo grep "audit_test" /etc/shadow
```

### Screenshots
- `1C-1_before_no_password.png`
  <img width="1366" height="720" alt="Screenshot 1C-1 BEFORE — etcshadow entry showing ! (no password" src="https://github.com/user-attachments/assets/31695c24-a54d-41d8-be76-63544bdf5768" />

- `1C-2_passwd_command.png`
  <img width="1366" height="720" alt="Screenshot 1C-2 passwd command setting the password" src="https://github.com/user-attachments/assets/23338556-34a4-486d-bcca-9a4bf68c64b2" />

- `1C-3_after_password_set.png`
  <img width="1366" height="720" alt="Screenshot 1C-3 AFTER — etcshadow entry showing hashed password $6$" src="https://github.com/user-attachments/assets/daf4622c-e18c-4f86-9ce0-55ae5b6f1824" />


### Before & After

| State | /etc/shadow Password Field |
|-------|---------------------------|
| Before | `!` — no password |
| After | `$6$...` — SHA-512 hash |

### Explanation

Running `passwd` sets a hashed password and updates the `/etc/shadow` entry. The change from `!` to a `$6$` hash confirms the account has been secured.

**Security Implication:** Passwords are never stored in plaintext. Linux uses salted SHA-512 hashing, making brute-force attacks computationally expensive. In production environments, remediation should always be followed by forcing a password change on first login using `chage -d 0 username`.
