# Commands Reference

**Assignment 1 – Linux User & Access Control**  
All commands used across both scenarios, organised by task.

---

## Scenario 1 – Password State Investigation & Remediation

### Part A – Create a User With No Password

```bash
# Create user without a password
sudo useradd audit_test

# Confirm the user exists
grep "audit_test" /etc/passwd
```

### Part B – Identify Users Without Passwords

```bash
# Inspect the shadow file (where password state is stored)
sudo cat /etc/shadow

# Check audit_test specifically
sudo grep "audit_test" /etc/shadow

# Filter all accounts with no active password (! or * or empty)
sudo grep -E "^[^:]+:(!|\*|::)" /etc/shadow

# Test non-privileged access (run as a normal user)
cat /etc/shadow
```

### Part C – Remediate: Set a Password

```bash
# BEFORE — confirm no password
sudo grep "audit_test" /etc/shadow

# Set the password
sudo passwd audit_test

# AFTER — confirm password is now set
sudo grep "audit_test" /etc/shadow
```

---

## Scenario 2 – DevSecOps User & Department Access Configuration

### Task 1 – Create All Users

```bash
# Using useradd (non-interactive, scriptable)
sudo useradd -m lateef
sudo useradd -m purity
sudo useradd -m arthur
sudo useradd -m paula
sudo useradd -m yaa
sudo useradd -m alex
sudo useradd -m habiba

# Using adduser (interactive, high-level)
sudo adduser aminat

# Service account — no login shell
sudo useradd -r -s /usr/sbin/nologin ci_runner
```

### Task 2 – Create Groups & Assign Users

```bash
# Create department groups
sudo groupadd dev_team
sudo groupadd sec_team
sudo groupadd qa_team
sudo groupadd ops_team

# Assign users to groups
sudo usermod -aG dev_team lateef
sudo usermod -aG dev_team arthur
sudo usermod -aG sec_team purity
sudo usermod -aG sec_team habiba
sudo usermod -aG qa_team paula
sudo usermod -aG qa_team yaa
sudo usermod -aG ops_team alex
sudo usermod -aG ops_team aminat

# Verify group assignments
getent group dev_team
getent group sec_team
getent group qa_team
getent group ops_team

# Check a specific user's groups
groups lateef
groups alex
```

### Task 3 – Passwordless User: Identify & Remediate

```bash
# BEFORE — confirm lateef has no password
sudo grep "lateef" /etc/shadow

# Set the password
sudo passwd lateef

# AFTER — confirm password is set
sudo grep "lateef" /etc/shadow
```

### Task 4 – Configure ci_runner With Non-Login Shell

```bash
# Verify the shell assigned to ci_runner
grep "ci_runner" /etc/passwd

# Attempt interactive login (should be denied)
su - ci_runner
```

### Task 5 – Grant Sudo Access to One ops_team User

```bash
# Grant sudo to alex only
sudo usermod -aG sudo alex        # Debian/Ubuntu
# sudo usermod -aG wheel alex     # RHEL/CentOS alternative

# Verify alex has sudo
sudo -l -U alex

# Confirm aminat does NOT have sudo
sudo -l -U aminat
```

### Task 6 – Remove User, Preserve Home Directory

```bash
# Remove yaa WITHOUT -r (preserves home directory)
sudo userdel yaa

# Confirm yaa is removed from the system
grep "yaa" /etc/passwd

# Confirm home directory still exists
ls -la /home/yaa
```

### Task 7 – Verification

```bash
# Check group memberships for all users
for user in lateef purity arthur paula yaa alex habiba aminat; do
  echo "--- $user ---"
  groups $user
done

# Inspect /etc/group directly
cat /etc/group | grep -E "dev_team|sec_team|qa_team|ops_team"

# Privilege boundary demonstration
sudo -l -U alex      # Has sudo
sudo -l -U lateef    # Does not have sudo
```

---

## Quick Reference – Key Files

| File | Purpose |
|------|---------|
| `/etc/passwd` | Stores user account info (username, UID, GID, shell) |
| `/etc/shadow` | Stores password hashes and password policy state |
| `/etc/group` | Stores group names, GIDs, and group members |


## Quick Reference – Password Field Symbols in /etc/shadow

| Symbol | Meaning | Risk Level |
|--------|---------|------------|
| `!` | Account locked / no password set | High |
| `*` | Login disabled (system accounts) | Low — intentional |
| `$6$...` | Valid SHA-512 hashed password | Normal |
| *(empty)* | No password — anyone can log in | Critical |
