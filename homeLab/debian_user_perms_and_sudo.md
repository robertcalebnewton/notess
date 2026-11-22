# Debian User Permissions & sudo — Maximum Security

## Philosophy: Least Privilege + Defense in Depth

Every process runs as some user. Every user has capabilities. Shrink that surface — no user should have more access than needed to do their job.

---

## 1. User Creation & Hygiene

```bash
adduser --disabled-password alice
```

- Prefer `adduser` (interactive, sets sane defaults) over `useradd`.
- `--disabled-password` forces SSH-key-only auth from day one.
- Every human admin gets **one personal account**. No shared logins.

```bash
# Lock unused or service accounts
usermod -L bob
passwd -l bob
```

**Service accounts** get no shell, no password, and home in `/var/empty` or `/nonexistent`:

```bash
adduser --system --no-create-home --shell /usr/sbin/nologin myapp
```

---

## 2. SSH Hardening (Relevant to User Access)

File: `/etc/ssh/sshd_config`

```
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
AuthenticationMethods publickey
AllowUsers alice bob
MaxAuthTries 3
LoginGraceTime 30
ClientAliveInterval 300
ClientAliveCountMax 0
```

- Root login is disabled. Period.
- No password auth means no brute-force against passwords.
- `AllowUsers` whitelist — if you're not listed, you don't exist.
- `MaxAuthTries 3` + `LoginGraceTime 30` limits unauthenticated hanging connections.

---

## 3. Sudo Architecture

### 3.1 The `sudo` Group Convention

```bash
usermod -aG sudo alice
```

Members of `sudo` get full admin access (via `%sudo ALL=(ALL:ALL) ALL`). Keep this group **small** (1–3 people).

### 3.2 `/etc/sudoers` — The Critical File

Always edit with `visudo`. It validates syntax before saving — a broken sudoers locks you out.

```bash
visudo
visudo -f /etc/sudoers.d/somename   # drop-in files are cleaner
```

**Structure:** `who where = (as_whom) what`

### 3.3 Global Defaults (maximum paranoia)

```
Defaults        env_reset, mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Defaults        passwd_tries=3
Defaults        timestamp_timeout=0
Defaults        timestamp_type=tty
Defaults        log_input, log_output
Defaults        logfile="/var/log/sudo.log"
Defaults        iolog_dir="/var/log/sudo-io/%u"
```

| Directive         | Why                                                          |
|-------------------|--------------------------------------------------------------|
| `timestamp_timeout=0` | Always ask for password — never cache.                    |
| `timestamp_type=tty`  | Each terminal needs its own re-auth.                     |
| `log_input, log_output` | Full session recording. Audit trail for everything. |
| `env_reset`       | Strips inherited environment variables (LD_PRELOAD attacks). |
| `mail_badpass`    | Alert on failed sudo attempts.                                |

### 3.4 Principle: Restrict by Role, Not User

```bash
# /etc/sudoers.d/ops
Cmnd_Alias  NETWORKING = /usr/sbin/ip, /usr/sbin/ss, /bin/ping, /usr/bin/mtr
Cmnd_Alias  PACKAGES   = /usr/bin/apt update, /usr/bin/apt upgrade, /usr/bin/apt full-upgrade
Cmnd_Alias  LOGS       = /bin/journalctl, /usr/bin/dmesg, /bin/tail, /bin/less

%ops    ALL=(root)  NETWORKING
%webops ALL=(root)  /usr/bin/systemctl reload nginx, /usr/bin/systemctl reload apache2
```

### 3.5 No NOPASSWD Unless You Really Mean It

```bash
# NEVER do this:
alice ALL=(ALL) NOPASSWD: ALL
```

`NOPASSWD` eliminates the only authentication check sudo has. Acceptable for CI/CD service accounts or specific non-interactive commands like:

```bash
# Acceptable pattern — passwordless for one scripted command only
deploy ALL=(root) NOPASSWD: /usr/local/bin/deploy.sh
```

---

## 4. File Permissions (DAC)

### 4.1 umask

`/etc/login.defs`: `UMASK 027` — new files get `750`/`640` by default.

### 4.2 Home Directories

```
chmod 750 /home/alice
chmod 700 /home/alice/.ssh
chmod 600 /home/alice/.ssh/authorized_keys
```

### 4.3 SUID/SGID — Hunt and Eliminate

SUID binaries let unprivileged users run as the file owner. Every one is a risk.

```bash
# Find all SUID/SGID files:
find / -perm /6000 -type f 2>/dev/null
```

Disable SUID on any binary your system doesn't need (e.g. `umount`, `mount`, `su` on single-user workstations via `chmod -s` or a `nosuid` mount).

### 4.4 Sticky Bit on Shared Writable Dirs

```
/tmp — drwxrwxrwt
```

Sticky bit (`chmod +t`) prevents users from deleting each other's files in `/tmp`.

---

## 5. Pluggable Authentication Modules (PAM)

### 5.1 Password Policy (`/etc/pam.d/common-password`)

```
password requisite pam_pwhistory.so remember=5 use_authok
password requisite pam_pwquality.so minlen=12 dcredit=-1 ucredit=-1 ocredit=-1 lcredit=-1
password sufficient pam_unix.so sha512 shadow use_authtok
```

### 5.2 Brute-Force Lockout (`/etc/pam.d/common-auth`)

```bash
# Add before the first `auth sufficient ...` line:
auth required pam_faillock.so preauth audit silent deny=5 unlock_time=900
auth [default=die] pam_faillock.so authfail audit deny=5 unlock_time=900
```

5 failed logins → 15-minute lockout.

### 5.3 Two-Factor Auth for sudo

```bash
# Install:
apt install libpam-google-authenticator
google-authenticator   # run as the user, scan QR into phone

# /etc/pam.d/sudo — add at top:
auth required pam_google_authenticator.so
```

Now every `sudo` call requires password + TOTP code.

---

## 6. Auditing

### 6.1 Sudo Logs

With `Defaults logfile`, `log_input`, `log_output`:

```bash
tail -f /var/log/sudo.log
# Full typed-input / output replay stored in /var/log/sudo-io/$USER/
sudo sudoreplay -l       # list recorded sessions
sudo sudoreplay ID       # replay a session
```

### 6.2 System Auditing (auditd)

```bash
apt install auditd

# Watch all root commands:
auditctl -w /bin/su -p x -k priv_esc
auditctl -w /usr/bin/sudo -p x -k priv_esc
auditctl -w /etc/sudoers -p wa -k sudoers_change
```

### 6.3 Who Was Here?

```bash
last          # login history
lastb         # failed logins
ausearch -k priv_esc | aureport -f  # auditd privilege escalation report
```

---

## 7. Service Accounts & Systemd Hardening

Run services with **dynamic users**, no shell, no login:

```ini
# /etc/systemd/system/myapp.service
[Service]
DynamicUser=yes
SupplementaryGroups=myapp-data
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
NoNewPrivileges=true
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
```

- `DynamicUser=yes` — creates an ephemeral UID, removed on stop.
- `CapabilityBoundingSet` — only grant the Linux capabilities the app actually needs.
- `NoNewPrivileges=true` — prevents the process from calling `execve` on a SUID binary.

---

## 8. Fail2ban for SSH

```ini
# /etc/fail2ban/jail.local
[sshd]
enabled   = true
maxretry  = 3
bantime   = 3600
findtime  = 600
```

Bans IPs after 3 failed SSH logins within 10 minutes.

---

## 9. Quick Security Checklist

- [ ] Root login disabled (`PermitRootLogin no`)
- [ ] SSH password auth disabled (`PasswordAuthentication no`)
- [ ] `AllowUsers` whitelist set
- [ ] `sudo` group has ≤ 3 members
- [ ] `timestamp_timeout=0` in sudoers
- [ ] `log_input, log_output` enabled
- [ ] No `NOPASSWD` for wildcard commands
- [ ] All home dirs `750`, `.ssh` `700`, `authorized_keys` `600`
- [ ] `faillock` configured with `deny=5`
- [ ] Unused SUID binaries removed
- [ ] `auditd` watches sudo and sudoers
- [ ] `fail2ban` active on SSH
- [ ] `UMASK 027` in `/etc/login.defs`
- [ ] `DynamicUser=yes` for new services where possible
