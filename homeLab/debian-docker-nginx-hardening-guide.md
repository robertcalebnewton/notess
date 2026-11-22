# Building a Hardened Debian 13 Host for a Containerized NGINX Web Server

### A Defense-in-Depth Methodology for Self-Hosted Web Infrastructure (2026 Threat Environment)

---

## Abstract

Self-hosted Linux web servers operate in a hostile environment. Since the Linux kernel project became its own CVE Numbering Authority in February 2024, it has published thousands of CVEs per year. High-profile events show the attack surface is the whole stack, not only the application: the `xz-utils` supply-chain backdoor (CVE-2024-3094), the OpenSSH "regreSSHion" race condition (CVE-2024-6387), the `nf_tables` privilege escalation used in ransomware campaigns (CVE-2024-1086), the `sudo` chroot escalation (CVE-2025-32463), and repeated container-escape flaws in `runc` (CVE-2024-21626; CVE-2025-31133, CVE-2025-52565, CVE-2025-52881).

This document gives a reproducible, layered method for deploying a static-content NGINX web server in a Docker container on Debian 13 ("Trixie"). It starts from an explicit threat model and then works through each layer: firmware and boot, the operating system, remote administration, the network, the container runtime, the container workload, TLS, the software supply chain, detection, and recovery. Every control is tied to a specific adversary technique. The design assumes **breach is a matter of when, not if**: each layer is built to limit the damage when the layer above it fails. The trade-offs that are still debated, especially rootless containers versus disabling unprivileged user namespaces, are discussed openly rather than hidden.

**Keywords:** Debian, Docker, rootless containers, NGINX, hardening, defense in depth, nftables, TLS 1.3, supply-chain security, NIST SP 800-190

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Threat Landscape (2024–2026)](#2-threat-landscape-20242026)
3. [Threat Model and Design Principles](#3-threat-model-and-design-principles)
4. [Reference Architecture](#4-reference-architecture)
5. [Phase I — Acquisition and Installation](#5-phase-i--acquisition-and-installation)
6. [Phase II — Operating System Hardening](#6-phase-ii--operating-system-hardening)
7. [Phase III — Secure Remote Administration (SSH)](#7-phase-iii--secure-remote-administration-ssh)
8. [Phase IV — Host Firewall (nftables)](#8-phase-iv--host-firewall-nftables)
9. [Phase V — Container Runtime: Docker Engine](#9-phase-v--container-runtime-docker-engine)
10. [Phase VI — The NGINX Workload](#10-phase-vi--the-nginx-workload)
11. [Phase VII — TLS and Certificate Lifecycle](#11-phase-vii--tls-and-certificate-lifecycle)
12. [Phase VIII — Software Supply-Chain Integrity](#12-phase-viii--software-supply-chain-integrity)
13. [Phase IX — Logging, Monitoring, and Detection](#13-phase-ix--logging-monitoring-and-detection)
14. [Phase X — Patch Management and Maintenance Cadence](#14-phase-x--patch-management-and-maintenance-cadence)
15. [Phase XI — Backup, Recovery, and Incident Response](#15-phase-xi--backup-recovery-and-incident-response)
16. [Verification and Validation](#16-verification-and-validation)
17. [Discussion: Limitations and Open Trade-offs](#17-discussion-limitations-and-open-trade-offs)
18. [Conclusion](#18-conclusion)
19. [References](#19-references)
20. [Appendix A — Deployment Checklist](#appendix-a--deployment-checklist)

---

## 1. Introduction

The common way to "host a website in Docker" is three commands: install Docker, run `docker run -p 80:80 nginx`, and open a port on the router. This works, but it leaves several weaknesses in place:

1. The Docker daemon runs as `root`, so any container escape becomes full host compromise.
2. Published ports skip host firewall front-ends such as UFW, because Docker writes its own packet-filter rules ahead of them.
3. The image is pulled by a mutable tag (`latest`), so what runs is neither reproducible nor auditable.
4. NGINX's master process starts as root inside the container, with the full default capability set.
5. Nothing detects compromise, and nothing guarantees recovery.

This guide replaces that pattern with an engineered system. The scope is deliberately narrow: **one Debian 13 host, serving static content through one NGINX container, reachable from the public Internet over HTTPS.** Dynamic back-ends, orchestration (Kubernetes), and multi-host deployments are out of scope. The principles carry over to them.

### 1.1 Conventions

| Placeholder | Meaning |
|---|---|
| `example.com` | Your registered domain |
| `admin` | Your human administrative account |
| `dockrun` | Unprivileged service account that owns the rootless Docker daemon |
| `192.168.10.0/24` | Your trusted management LAN (adjust to your network) |
| `#` prompt | Run as root (via `sudo`) |
| `$` prompt | Run as an unprivileged user |

> **Version note.** Commands target Debian 13 (Linux 6.12 LTS kernel, OpenSSH 10.x, systemd 257, APT 3.0) and a current Docker Engine release from Docker's official repository. Software moves quickly: check every version-specific claim against the upstream documentation listed in §19 before relying on it.

---

## 2. Threat Landscape (2024–2026)

The claim that "Linux is compromised weekly" is best understood as a statement about **vulnerability disclosure tempo and exploitation latency**, not about one catastrophic flaw. Three trends define the current environment.

### 2.1 Vulnerability volume

Since February 2024 the kernel CNA assigns CVEs to essentially every bug-fix commit that may have security impact. The volume is now so large that "read every CVE and decide" no longer works for small operators. The practical answer is **architectural**: reduce reachable attack surface, contain the blast radius, and **patch on a fixed, automated cadence** instead of patching reactively.

### 2.2 Representative incidents

| Identifier | Component | Class | Lesson for this design |
|---|---|---|---|
| CVE-2024-3094 | `xz-utils` / `liblzma` | Supply-chain backdoor targeting `sshd` | Trust in upstream is not absolute. Limit SSH exposure, and prefer stable distributions that adopt new upstream releases slowly. |
| CVE-2024-6387 ("regreSSHion") | OpenSSH `sshd` | Pre-auth remote code execution (signal-handler race) | **Never expose SSH to the Internet** unless you must. Gate it behind a VPN or allow-list. |
| CVE-2024-1086 | Kernel `nf_tables` | Local privilege escalation; listed in CISA KEV and reported in ransomware use | Kernel LPEs turn any foothold into root. Unprivileged user namespaces widen this surface (see §9.2). |
| CVE-2024-21626 ("Leaky Vessels") | `runc` | Container escape via leaked file descriptor | Container isolation is a mitigation, not a boundary. Run the runtime unprivileged. |
| CVE-2025-31133, CVE-2025-52565, CVE-2025-52881 | `runc` | Container escape via mount/procfs races | Container escapes keep recurring. Rootless mode and user-namespace remapping turn "escape" into "escape to an unprivileged user". |
| CVE-2025-32463 | `sudo` (`--chroot`) | Local privilege escalation | Even core administration tools fail. Minimize sudo rules and audit their use. |

### 2.3 Dominant adversary behaviors against small web hosts

Mapped to MITRE ATT&CK for Enterprise:

- **T1190 — Exploit Public-Facing Application**: automated scanning of every IPv4 address within hours of a CVE being published.
- **T1110 — Brute Force**: password spraying against SSH, continuously.
- **T1068 — Exploitation for Privilege Escalation**: kernel and SUID-binary LPEs after the initial foothold.
- **T1611 — Escape to Host**: breaking out of containers to the host.
- **T1195 — Supply Chain Compromise**: poisoned images, packages, or build tooling.
- **T1505.003 — Web Shell**: persistence by writing to the web root.
- **T1496 — Resource Hijacking**: crypto-mining, the most common goal on compromised Linux servers.

---

## 3. Threat Model and Design Principles

### 3.1 Assets

1. **Integrity of served content.** Defacement or malware injection damages visitors and reputation.
2. **Availability** of the site.
3. **Confidentiality of the TLS private key.** Theft enables impersonation.
4. **The host itself.** It is a pivot point into the home or office LAN, and compute that can be hijacked.

### 3.2 Adversaries

| Adversary | Capability | In scope? |
|---|---|---|
| Opportunistic botnets / scanners | Automated exploitation of known CVEs, credential stuffing | **Yes — primary** |
| Skilled targeted attacker | Chaining n-day exploits, custom tooling | Yes — design aims to make attacks costly and detectable |
| Upstream supply-chain compromise | Malicious image or package | Partially — mitigated by pinning, scanning, and verification |
| Nation-state with 0-days / physical access | Firmware implants, hardware attacks | **No** — beyond the means of this design |

### 3.3 Design principles

1. **Attack-surface minimization.** Every installed package, listening socket, and capability is a liability until justified.
2. **Least privilege.** No component runs with more privilege than its function needs: NGINX gets no capabilities, the Docker daemon is not root, and the human administrator has no standing root shell.
3. **Defense in depth.** Assume each layer fails on its own. Compromise of NGINX should not yield the host, and compromise of the host should be detectable.
4. **Immutability.** The running container's filesystem is read-only. Changes happen by rebuilding and redeploying, never by editing in place.
5. **Reproducibility.** All configuration is text, kept under version control, so the system can be **rebuilt rather than cleaned** after an incident.
6. **Automated hygiene over heroic response.** Patching, renewal, and image refresh run on timers, not on memory.

### 3.4 Control-to-threat mapping

| Layer | Primary controls | Threats mitigated |
|---|---|---|
| Boot | UEFI Secure Boot, kernel lockdown, LUKS2 | Bootkits, offline tampering, disk theft |
| OS | Minimal install, sysctl hardening, module blacklist, AppArmor, auto-updates | T1068 |
| Admin access | Key-only / FIDO2 SSH, no root login, VPN or LAN allow-list | T1110, CVE-2024-6387 class |
| Network | Default-deny nftables, upstream edge firewall | T1190 (surface reduction), lateral movement |
| Runtime | Rootless Docker, seccomp, no-new-privileges | T1611 |
| Workload | Unprivileged NGINX, `cap_drop: ALL`, read-only rootfs, resource limits | T1505.003, T1496 |
| Supply chain | Digest pinning, vulnerability scanning, signature verification | T1195 |
| Detection | auditd, AIDE, centralized logs, Lynis | Dwell time reduction |
| Recovery | Config-as-code, offline backups, rebuild procedure | All (resilience) |

---

## 4. Reference Architecture

```
                 Internet
                    │
                    ▼
     ┌─────────────────────────────┐
     │  Edge firewall / router      │  e.g. OpenBSD pf gateway
     │  • forwards ONLY tcp/80,443  │  • no SSH forwarded
     │  • host on isolated VLAN/DMZ │
     └──────────────┬──────────────┘
                    │
     ┌──────────────▼──────────────────────────────────────────┐
     │  Debian 13 host  (Secure Boot · LUKS2 · AppArmor)        │
     │                                                          │
     │  nftables: default-deny inbound                          │
     │    allow 80,443 from any; 22 from mgmt LAN / WireGuard   │
     │                                                          │
     │  ┌────────────────────────────────────────────────────┐  │
     │  │ user "dockrun" (UID ≥1000, no sudo, no SSH login)  │  │
     │  │  └─ rootless dockerd (user namespace)              │  │
     │  │      └─ container "web-nginx"                      │  │
     │  │          • UID 101 → host subordinate UID          │  │
     │  │          • cap_drop ALL, no-new-privileges         │  │
     │  │          • read-only rootfs, tmpfs /tmp            │  │
     │  │          • content, config, certs mounted :ro      │  │
     │  │          • listens 8080/8443 → host 80/443         │  │
     │  └────────────────────────────────────────────────────┘  │
     │                                                          │
     │  certbot (host, root) → copies certs → reload container  │
     │  auditd · AIDE · journald → remote log sink              │
     └──────────────────────────────────────────────────────────┘
```

**Key property:** for an attacker who gets code execution inside NGINX, the path to root on the host is
*NGINX worker (UID 101, no capabilities)* → *container escape* → *unprivileged `dockrun` user* → *kernel LPE*. That is **at least three independent exploits** instead of one.

### 4.1 Alternative edge: outbound tunnel

Residential connections often use CGNAT, change IP addresses, or have ISP terms that forbid servers. An **outbound-only tunnel** (for example, Cloudflare Tunnel via `cloudflared`) removes all inbound port-forwarding and hides the origin IP. The trade-off is that TLS is terminated by a third party who can see plaintext traffic. This guide assumes direct exposure. Under the tunnel model, §8 simplifies to "allow no inbound Internet traffic at all".

---

## 5. Phase I — Acquisition and Installation

### 5.1 Verify the installation media

Never install from an image you have not verified. Download the `netinst` ISO, the `SHA512SUMS` file, and `SHA512SUMS.sign` from `https://www.debian.org/CD/`.

```bash
$ gpg --keyserver keyring.debian.org --recv-keys <Debian CD signing key ID>
$ gpg --verify SHA512SUMS.sign SHA512SUMS          # must report "Good signature"
$ sha512sum --ignore-missing -c SHA512SUMS         # must report "OK" for your ISO
```

Check the key fingerprint against the list published at `https://www.debian.org/CD/verify`, reached over a separate channel from the download. A signature check against an attacker-supplied key proves nothing.

### 5.2 Firmware configuration

| Setting | Value | Rationale |
|---|---|---|
| UEFI mode | Enabled (no CSM/legacy) | Required for Secure Boot |
| Secure Boot | **Enabled** | Debian ships a Microsoft-signed `shim`. With Secure Boot on, the kernel enters **lockdown (integrity) mode**, which blocks unsigned modules and `/dev/mem` access even for root. |
| Firmware admin password | Set | Prevents trivial boot-order changes |
| Boot order | Internal disk only | Prevents booting from attacker media |
| TPM 2.0 | Enabled | Allows measured-boot disk unlock (§5.4) |
| Unused devices (Wi-Fi, Bluetooth, audio) | Disabled | Surface reduction |

### 5.3 Partitioning

Separating filesystems lets you apply restrictive mount options and keeps log or container growth from exhausting the root filesystem (a denial-of-service vector).

| Mount point | Size (guide) | Options | Purpose |
|---|---|---|---|
| `/boot/efi` | 512 MiB | `umask=0077` | EFI system partition |
| `/boot` | 1 GiB | `nodev,nosuid,noexec` | Unencrypted kernel/initramfs |
| LUKS2 → LVM | rest of disk | — | Encrypted container |
| ├ `/` | 20 GiB | defaults | Root |
| ├ `/var` | 10 GiB | `nodev,nosuid` | Logs, APT cache |
| ├ `/var/log` | 5 GiB | `nodev,nosuid,noexec` | Bounded log growth |
| ├ `/var/log/audit` | 2 GiB | `nodev,nosuid,noexec` | Protects audit trail from log flooding |
| ├ `/home` | 30 GiB+ | `nodev,nosuid` | Holds rootless Docker data (`~dockrun/.local/share/docker`) |
| └ swap | 2–4 GiB | — | Encrypted with everything else |

Debian 13 mounts `/tmp` as **tmpfs by default** (with `nosuid,nodev`). To also forbid execution there, which defeats many "download to `/tmp` and run" payloads, see §6.6.

### 5.4 Full-disk encryption

Choose **LUKS2** in the guided "encrypted LVM" partitioner. Modern Debian defaults to the memory-hard **argon2id** key-derivation function. Confirm after installation:

```bash
# cryptsetup luksDump /dev/<luks-partition> | grep -E 'Version|PBKDF'
```

**Headless unlock trade-off.** A server that waits for a passphrase at boot cannot recover unattended from a power cut. Your options:

| Method | Security | Availability |
|---|---|---|
| Passphrase at console | Highest | Poor (needs physical presence) |
| `dropbear-initramfs` (SSH into initramfs to unlock) | High | Good (remote, manual) |
| `clevis` + TPM2 bound to PCR 7 (Secure Boot state) | Moderate: protects against disk theft, not against theft of the whole machine | Excellent (automatic) |

For a home or small-office server, **TPM2 binding with a backup passphrase in a second LUKS keyslot** is a reasonable balance.

### 5.5 Package selection

In `tasksel`, select **only** "SSH server" and "standard system utilities". **Do not install a desktop environment or web server** on the host. NGINX lives only in the container.

---

## 6. Phase II — Operating System Hardening

### 6.1 Administrative identity

Do not set a root password during installation. Debian then locks the root account and grants `sudo` to the first user, so every privileged action is logged against a person.

```bash
# groupadd --system sshusers
# usermod -aG sudo,sshusers admin
# passwd -S root            # should show "L" (locked)
```

### 6.2 APT sources and update automation

Debian 13 uses the deb822 source format. Confirm the **security** suite is present in `/etc/apt/sources.list.d/debian.sources`:

```
Types: deb
URIs: https://deb.debian.org/debian
Suites: trixie trixie-updates
Components: main non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb
URIs: https://security.debian.org/debian-security
Suites: trixie-security
Components: main non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

Install automatic security updates and the supporting tools:

```bash
# apt update && apt full-upgrade
# apt install unattended-upgrades apt-listchanges needrestart debsecan
# dpkg-reconfigure -plow unattended-upgrades
```

Create `/etc/apt/apt.conf.d/52unattended-upgrades-local`:

```
Unattended-Upgrade::Origins-Pattern {
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Docker,label=Docker CE";
};
Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
// Kernel fixes only take effect after a reboot. Debian has no free livepatch service.
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "04:30";
```

> **Rationale for automatic reboots.** Given the kernel CVE tempo described in §2.1, a host that installs kernel updates but never reboots is running vulnerable code. For a single static site, a nightly maintenance window of under a minute is an acceptable availability cost.

### 6.3 Remove unneeded software

```bash
# apt purge --autoremove exim4-base exim4-config 2>/dev/null || true
# ss -tulpn          # inventory: only sshd should be listening at this stage
```

### 6.4 Kernel parameters

Create `/etc/sysctl.d/90-hardening.conf`:

```ini
# --- Information leakage ---
kernel.kptr_restrict = 2
kernel.dmesg_restrict = 1
kernel.perf_event_paranoid = 3

# --- Exploit mitigation ---
kernel.unprivileged_bpf_disabled = 1
net.core.bpf_jit_harden = 2
kernel.yama.ptrace_scope = 2
kernel.kexec_load_disabled = 1
kernel.sysrq = 0
fs.suid_dumpable = 0
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
fs.protected_fifos = 2
fs.protected_regular = 2

# --- Network: IPv4 ---
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1

# --- Network: IPv6 ---
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_source_route = 0

# --- Rootless Docker (see §9.2): let unprivileged users bind 80 and up ---
net.ipv4.ip_unprivileged_port_start = 80
```

```bash
# sysctl --system
```

> **Caution on `net.ipv4.ip_unprivileged_port_start`.** This lets *any* local user bind ports 80–1023. On a single-purpose server with no untrusted local users the risk is negligible. The alternative, `setcap cap_net_bind_service=ep` on the `rootlesskit` binary, is silently lost every time the package upgrades, which makes it a maintenance trap.

### 6.5 Disable unused kernel modules

Many kernel LPEs sit in rarely used protocol or filesystem code that loads automatically on demand. Create `/etc/modprobe.d/90-hardening.conf`:

```
# Uncommon network protocols
install dccp /bin/false
install sctp /bin/false
install rds /bin/false
install tipc /bin/false
install n-hdlc /bin/false
install ax25 /bin/false
install netrom /bin/false
install x25 /bin/false
install rose /bin/false
install decnet /bin/false
install appletalk /bin/false
install atm /bin/false
# Uncommon filesystems
install cramfs /bin/false
install freevxfs /bin/false
install jffs2 /bin/false
install hfs /bin/false
install hfsplus /bin/false
install udf /bin/false
# Physical attack surface (headless server)
install usb-storage /bin/false
install firewire-core /bin/false
install thunderbolt /bin/false
```

```bash
# update-initramfs -u
```

### 6.6 Mount hardening

```bash
# systemctl edit tmp.mount
```

```ini
[Mount]
Options=mode=1777,strictatime,nosuid,nodev,noexec,size=50%%,nr_inodes=1m
```

Add to `/etc/fstab`:

```
tmpfs  /dev/shm  tmpfs  defaults,nosuid,nodev,noexec  0 0
proc   /proc     proc   defaults,hidepid=invisible    0 0
```

`hidepid=invisible` stops users from seeing other users' processes, which limits reconnaissance after a foothold. Test after a reboot: a few tools (some older `systemd-logind` setups, `polkit`) can misbehave with it. Remove it if they do.

### 6.7 Mandatory access control

AppArmor is enabled by default on Debian. Install the utilities and confirm:

```bash
# apt install apparmor-utils apparmor-profiles apparmor-profiles-extra
# aa-status        # all loaded profiles should be in "enforce" mode
```

### 6.8 Restrict `su` and `sudo`

```bash
# dpkg-statoverride --update --add root sudo 4750 /bin/su
```

Keep `/etc/sudoers.d/` empty except for deliberate, minimal rules. **Do not** add `NOPASSWD`. After CVE-2025-32463, confirm the installed `sudo` is patched (`sudo -V | head -1`) and avoid any rule that uses the `CHROOT=` option.

### 6.9 Time synchronization

Accurate time is a precondition for TLS validation, log correlation, and certificate renewal. For authenticated time, replace `systemd-timesyncd` with `chrony` using **Network Time Security (NTS, RFC 8915)**:

```bash
# apt install chrony
```

In `/etc/chrony/sources.d/nts.sources`:

```
server time.cloudflare.com iburst nts
server nts.netnod.se iburst nts
```

```bash
# systemctl restart chrony && chronyc -N authdata    # "NTS" mode should appear
```

---

## 7. Phase III — Secure Remote Administration (SSH)

### 7.1 Exposure policy

**SSH must not be reachable from the Internet.** CVE-2024-6387 showed that even OpenSSH, one of the most heavily audited network daemons, can carry a pre-authentication remote-code-execution flaw. Reach SSH only from:

- the management LAN (enforced in §8), or
- a WireGuard tunnel terminating on the edge firewall or the host.

### 7.2 Key material

On your **workstation**, prefer a hardware-backed FIDO2 key (YubiKey, SoloKey, etc.). The private key cannot be extracted, and every authentication needs a physical touch:

```bash
$ ssh-keygen -t ed25519-sk -O resident -O verify-required -C "admin@workstation"
# Fallback without a hardware token:
$ ssh-keygen -t ed25519 -a 100 -C "admin@workstation"
$ ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub admin@server
```

### 7.3 Daemon configuration

Debian's `sshd_config` includes `/etc/ssh/sshd_config.d/*.conf` **first**, and OpenSSH uses the **first value** it finds for each keyword, so a drop-in file overrides the defaults. Create `/etc/ssh/sshd_config.d/10-hardening.conf`:

```
# --- Authentication ---
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitEmptyPasswords no
PubkeyAuthentication yes
AuthenticationMethods publickey
AllowGroups sshusers
MaxAuthTries 3
LoginGraceTime 20
MaxStartups 10:30:60

# --- Feature reduction ---
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
AllowStreamLocalForwarding no
PermitTunnel no
PermitUserEnvironment no
GatewayPorts no

# --- Session hygiene ---
ClientAliveInterval 300
ClientAliveCountMax 2

# --- Cryptography (post-quantum hybrid key exchange first) ---
KexAlgorithms mlkem768x25519-sha256,sntrup761x25519-sha512@openssh.com,curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
HostKeyAlgorithms ssh-ed25519,sk-ssh-ed25519@openssh.com,rsa-sha2-512
PubkeyAcceptedAlgorithms ssh-ed25519,sk-ssh-ed25519@openssh.com
```

**Post-quantum note.** `mlkem768x25519-sha256` combines the NIST-standardized ML-KEM (FIPS 203) with X25519. Recorded traffic stays secure even against a future quantum adversary ("harvest now, decrypt later"), as long as *either* primitive holds. It is the default in OpenSSH 10.

**Built-in rate limiting.** OpenSSH 9.8 and later include `PerSourcePenalties`, which penalizes source addresses that repeatedly fail authentication or crash the pre-auth process. Together with key-only authentication, this makes `fail2ban` for SSH largely redundant.

Remove the weak host keys and regenerate moduli if desired:

```bash
# rm -f /etc/ssh/ssh_host_ecdsa_key*      # keep ed25519 (and rsa only if legacy clients need it)
# sshd -t && systemctl reload ssh
```

> **Lock-out prevention.** Keep your current session open and test a **new** login in a second terminal before closing the first.

Audit the result from your workstation with `ssh-audit` (§16).

---

## 8. Phase IV — Host Firewall (nftables)

### 8.1 The Docker/firewall interaction problem

Rootful Docker publishes ports by inserting DNAT rules into the kernel packet filter. Traffic to published ports is forwarded, not delivered locally, so it **never passes through the host `INPUT` chain** where UFW and similar tools filter. Many operators think port 8080 is "blocked by UFW" while it is open to the world.

The **rootless** design chosen in §9 removes this problem. `rootlesskit` binds published ports as an ordinary userspace socket in the host network namespace, so they are subject to the normal `input` hook like any other service.

### 8.2 Ruleset

Install and write `/etc/nftables.conf`:

```bash
# apt install nftables
```

```nft
#!/usr/sbin/nft -f
# Host firewall — rootless-Docker web server
flush ruleset

table inet filter {

    set mgmt_v4 {
        type ipv4_addr; flags interval;
        elements = { 192.168.10.0/24, 10.200.0.0/24 }   # LAN, WireGuard
    }

    chain input {
        type filter hook input priority filter; policy drop;

        ct state invalid drop
        ct state { established, related } accept
        iif "lo" accept

        # ICMPv6 is mandatory for IPv6 to function (ND, PMTUD)
        meta l4proto ipv6-icmp icmpv6 type {
            destination-unreachable, packet-too-big, time-exceeded,
            parameter-problem, nd-router-advert, nd-neighbor-solicit,
            nd-neighbor-advert
        } accept
        meta l4proto ipv6-icmp icmpv6 type echo-request limit rate 5/second accept

        # ICMPv4 essentials
        meta l4proto icmp icmp type { destination-unreachable, time-exceeded, parameter-problem } accept
        meta l4proto icmp icmp type echo-request limit rate 5/second accept

        # SSH — management networks only
        tcp dport 22 ip saddr @mgmt_v4 ct state new limit rate 15/minute accept

        # Web — public
        tcp dport { 80, 443 } ct state new accept

        # Visibility into what is being dropped (rate-limited)
        limit rate 10/minute log prefix "nft-in-drop: " flags all
    }

    chain forward {
        type filter hook forward priority filter; policy drop;
        # Rootless Docker does not use kernel forwarding. If you switch to
        # rootful Docker you must NOT keep this policy — see §8.3.
    }

    chain output {
        type filter hook output priority filter; policy accept;
        # Egress filtering is a valuable advanced control: a web server
        # needs DNS, NTP, APT, the ACME CA, and the container registry.
        # Restricting the rest blocks reverse shells and mining pools.
    }
}
```

```bash
# nft -c -f /etc/nftables.conf      # syntax check
# systemctl enable --now nftables
# nft list ruleset
```

### 8.3 If you choose rootful Docker instead

- Do **not** use `flush ruleset`, which wipes Docker's own tables. Flush only `table inet filter`.
- Do not set the `forward` policy to drop without explicitly allowing Docker bridge traffic.
- Put access restrictions for published ports in the **`DOCKER-USER`** chain, which Docker reserves for administrator rules and evaluates before its own.
- Recent Docker Engine releases offer an nftables firewall backend. Check your version's documentation for its maturity before relying on it.

### 8.4 Edge firewall

On the upstream router or firewall, forward **only TCP 80 and 443** to the host. Ideally place the host in an isolated network segment (DMZ/VLAN) that **cannot open connections to the internal LAN**. If the web server is compromised, it must not become a beachhead into household or corporate devices.

---

## 9. Phase V — Container Runtime: Docker Engine

### 9.1 Installation from the upstream repository

Debian's own `docker.io` package is reasonable and patched by the Debian Security Team. Docker's repository ships fixes (particularly for `runc` and `containerd`) faster and includes the rootless extras. This guide uses the upstream repository.

```bash
# apt install ca-certificates curl
# install -m 0755 -d /etc/apt/keyrings
# curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
# gpg --show-keys /etc/apt/keyrings/docker.asc
#   → compare the fingerprint with Docker's published value
#     (9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88)
# chmod a+r /etc/apt/keyrings/docker.asc

# tee /etc/apt/sources.list.d/docker.sources >/dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# apt update
# apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin \
              docker-compose-plugin docker-ce-rootless-extras \
              uidmap dbus-user-session slirp4netns systemd-container
```

### 9.2 Choosing an isolation model: a real trade-off

This is the most important architectural decision in the guide, and the security community has **not settled it**.

| Model | How it works | Benefit | Cost |
|---|---|---|---|
| **A. Rootless Docker** (chosen) | Daemon and containers run as an unprivileged user inside a user namespace | Daemon compromise or container escape yields an **unprivileged** user, not root. Ports obey the host firewall. | Requires **unprivileged user namespaces**, which let any local user gain `CAP_NET_ADMIN` etc. *inside their namespace*. That widens the kernel attack surface: many `nf_tables` LPEs, including public CVE-2024-1086 exploits, rely on it. No AppArmor for containers. |
| **B. Rootful Docker + `userns-remap`** | Root daemon; container root remapped to a subordinate UID | Lets you **disable** unprivileged user namespaces system-wide (`kernel.unprivileged_userns_clone=0`). AppArmor applies. | The daemon remains root. A daemon or `containerd` vulnerability yields root directly. Published ports bypass `INPUT` (§8.1). |
| **C. Rootless Podman** | Daemonless, rootless, systemd-native | Same as A, with no long-running daemon | Same user-namespace cost as A; different tooling |

**Analysis.** For a host with **no untrusted local users**, the extra kernel surface from unprivileged user namespaces matters only *after* an attacker already has code execution. Model A's advantage applies at exactly that point: the attacker lands as an unprivileged user and must *still* find a kernel LPE. Model B denies the attacker user namespaces but gives them a root-owned daemon socket and root-owned runtime as targets. This guide judges **Model A** to have the better expected outcome for a single-tenant web server and pairs it with aggressive kernel patching (§14) and module blacklisting (§6.5) to address its main weakness. A multi-user host would likely tip the balance toward Model B.

### 9.3 Configure rootless Docker

Disable the system-wide root daemon:

```bash
# systemctl disable --now docker.service docker.socket
# rm -f /var/run/docker.sock
```

Create the dedicated service account. It is **not** in `sudo`, `docker`, or `sshusers`:

```bash
# useradd --create-home --shell /bin/bash --comment "Rootless Docker runtime" dockrun
# grep dockrun /etc/subuid /etc/subgid      # must show a 65536-ID subordinate range
# loginctl enable-linger dockrun            # daemon survives without a login session
```

Delegate cgroup v2 controllers so resource limits (`cpus`, `mem_limit`, `pids_limit`) are enforceable in rootless mode. Create `/etc/systemd/system/user@.service.d/delegate.conf`:

```ini
[Service]
Delegate=cpu cpuset io memory pids
```

```bash
# systemctl daemon-reload
```

Enter a proper systemd user session as `dockrun`. Plain `su` does not create the `XDG_RUNTIME_DIR` and D-Bus session that rootless mode needs:

```bash
# machinectl shell dockrun@
$ dockerd-rootless-setuptool.sh install
```

**Preserve client source IPs.** The default rootlesskit port driver rewrites the source address of every incoming connection to an internal one, which breaks NGINX logging and per-IP rate limiting. Use the `slirp4netns` port driver:

```bash
$ mkdir -p ~/.config/systemd/user/docker.service.d
$ cat > ~/.config/systemd/user/docker.service.d/override.conf <<'EOF'
[Service]
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_NET=slirp4netns"
Environment="DOCKERD_ROOTLESS_ROOTLESSKIT_PORT_DRIVER=slirp4netns"
EOF
```

Daemon hardening, `~/.config/docker/daemon.json`:

```json
{
  "icc": false,
  "no-new-privileges": true,
  "live-restore": true,
  "log-driver": "local",
  "log-opts": { "max-size": "20m", "max-file": "5" },
  "default-ulimits": { "nofile": { "Name": "nofile", "Soft": 4096, "Hard": 8192 } }
}
```

| Key | Effect |
|---|---|
| `icc: false` | Containers on the default bridge cannot talk to each other |
| `no-new-privileges` | Default for all containers: blocks privilege gain via setuid binaries |
| `live-restore` | Containers keep running across daemon restarts or upgrades |
| `log-driver: local` + limits | Bounded, compressed logs. Prevents disk exhaustion. |

```bash
$ systemctl --user daemon-reload
$ systemctl --user enable --now docker
$ echo 'export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock' >> ~/.bashrc
$ docker info --format '{{.SecurityOptions}}'
#   → expect: [name=seccomp,profile=builtin name=rootless name=cgroupns]
```

---

## 10. Phase VI — The NGINX Workload

### 10.1 Base image selection

| Option | Process UID | Notes |
|---|---|---|
| `nginx:<ver>` (Docker Official Image) | Master runs as root, workers as `nginx` | Needs `CHOWN`, `SETUID`, `SETGID`, `NET_BIND_SERVICE` |
| **`nginxinc/nginx-unprivileged:<ver>-alpine`** (chosen) | All processes UID 101 | Maintained by the NGINX project. Listens on 8080. Needs **zero** capabilities. Small Alpine base. |
| Distroless / custom static build | Arbitrary | Smallest surface; highest maintenance burden |

Use the **stable** branch (even minor version) unless you need a mainline feature. Confirm the current tag and repository name on Docker Hub before deploying.

### 10.2 Project layout (owned by `dockrun`)

```
/home/dockrun/web/
├── compose.yaml
├── .env                  # pinned image digest
├── conf/
│   └── site.conf         # mounted read-only
├── html/                 # site content, mounted read-only
│   └── index.html
└── certs/                # populated by certbot deploy hook (§11)
```

Keep the directory in a Git repository (excluding `certs/`) so the deployment is reproducible.

### 10.3 Pin the image by digest

A tag such as `1.28-alpine` is a **mutable pointer**: whoever controls the registry account can move it. A digest is a cryptographic hash of the content.

```bash
$ docker pull docker.io/nginxinc/nginx-unprivileged:1.28-alpine
$ docker image inspect --format '{{index .RepoDigests 0}}' \
      docker.io/nginxinc/nginx-unprivileged:1.28-alpine
#   → docker.io/nginxinc/nginx-unprivileged@sha256:3f1b…
```

`.env`:

```
NGINX_IMAGE=docker.io/nginxinc/nginx-unprivileged:1.28-alpine@sha256:<digest>
```

### 10.4 NGINX configuration

`conf/site.conf` is included inside the image's `http {}` block:

```nginx
# ---------- http-level ----------
server_tokens off;

# Abuse controls
limit_req_zone  $binary_remote_addr zone=perip:10m rate=10r/s;
limit_conn_zone $binary_remote_addr zone=connperip:10m;

# Slowloris / resource-exhaustion mitigation
client_body_timeout    10s;
client_header_timeout  10s;
keepalive_timeout      15s;
send_timeout           10s;
client_max_body_size   1k;          # static site: accept no request bodies
large_client_header_buffers 4 8k;

# ---------- HTTP (8080 → host 80): ACME + redirect only ----------
server {
    listen 8080 default_server;
    listen [::]:8080 default_server;
    server_name _;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/acme;
        default_type text/plain;
    }

    location = /healthz {
        allow 127.0.0.1;
        deny  all;
        access_log off;
        return 200 "ok\n";
    }

    location / {
        return 301 https://example.com$request_uri;
    }
}

# ---------- HTTPS (8443 → host 443) ----------
server {
    listen 8443 ssl default_server;
    listen [::]:8443 ssl default_server;
    http2 on;
    server_name example.com www.example.com;

    # --- TLS: Mozilla "intermediate" profile ---
    ssl_certificate     /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;
    # Hybrid post-quantum group first, if the image's OpenSSL supports it (≥ 3.5):
    #   ssl_ecdh_curve X25519MLKEM768:X25519:prime256v1:secp384r1;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;
    ssl_session_cache shared:TLS:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;           # preserves forward secrecy across restarts

    # --- Security headers ---
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
    add_header Content-Security-Policy "default-src 'self'; img-src 'self' data:; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(), payment=(), usb=()" always;
    add_header Cross-Origin-Opener-Policy "same-origin" always;
    add_header Cross-Origin-Resource-Policy "same-origin" always;

    # --- Abuse controls ---
    limit_req  zone=perip burst=20 nodelay;
    limit_conn connperip 20;
    limit_req_status  429;
    limit_conn_status 429;

    # Static site: allow only safe methods
    if ($request_method !~ ^(GET|HEAD)$) { return 405; }

    root  /usr/share/nginx/html;
    index index.html;

    # RFC 9116 security.txt is the only dotfile path allowed
    location = /.well-known/security.txt { try_files $uri =404; }
    location ~ /\. { deny all; return 404; }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Configuration notes.**

- **OCSP stapling is deliberately omitted.** Let's Encrypt ended its OCSP service in 2025 and no longer puts OCSP URLs in certificates, so `ssl_stapling` would do nothing or log errors. Revocation now relies on CRLs.
- **`add_header` inheritance pitfall.** If any `location` block declares its own `add_header`, it **drops all** server-level headers for that location. Repeat the full set, or use an `include` snippet.
- **CSP** is the strongest browser-side defense against injected content. Tighten it to your site's needs and test with the browser console. Use `Content-Security-Policy-Report-Only` while tuning.
- **HSTS `preload`** is omitted because preload-list submission is hard to reverse. Add it only after the domain has been HTTPS-only for a long time.

### 10.5 Compose definition

`compose.yaml`:

```yaml
name: web

services:
  nginx:
    image: ${NGINX_IMAGE}
    container_name: web-nginx
    restart: unless-stopped
    user: "101:101"

    # --- Privilege minimization ---
    cap_drop: [ALL]
    security_opt:
      - no-new-privileges:true
      # seccomp: Docker's default profile is applied automatically.
      # AppArmor is not supported in rootless mode (see §17).

    # --- Immutability ---
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,nodev,size=64m,mode=1777

    volumes:
      - ./conf/site.conf:/etc/nginx/conf.d/default.conf:ro
      - ./html:/usr/share/nginx/html:ro
      - ./certs:/etc/nginx/certs:ro
      - /srv/acme:/var/www/acme:ro

    # --- Exposure ---
    ports:
      - "80:8080"
      - "443:8443"

    # --- Resource exhaustion / cryptojacking containment ---
    cpus: "1.0"
    mem_limit: 256m
    memswap_limit: 256m
    pids_limit: 100

    healthcheck:
      test: ["CMD", "wget", "-q", "-O", "/dev/null", "http://127.0.0.1:8080/healthz"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s

    logging:
      driver: local
      options: { max-size: "20m", max-file: "5" }
```

**Why each control matters if an attacker gets code execution in NGINX:**

| Control | What the attacker cannot do |
|---|---|
| `user: 101` + rootless | Act as root in the container; root-in-container is in any case only `dockrun` on the host |
| `cap_drop: ALL` | Raw sockets, `chown`, `setuid`, mount, packet sniffing |
| `no-new-privileges` | Escalate through any setuid binary in the image |
| `read_only` + `:ro` mounts | Write a web shell, alter content, or install persistence (T1505.003) |
| `tmpfs … noexec` | Download and run a payload in the only writable location |
| `cpus` / `mem_limit` / `pids_limit` | Run a profitable miner or fork-bomb the host (T1496) |
| Default seccomp | Call ~40+ dangerous syscalls (e.g. `kexec_load`, `open_by_handle_at`, `bpf`) |

---

## 11. Phase VII — TLS and Certificate Lifecycle

### 11.1 Context: shrinking certificate lifetimes

Under CA/Browser Forum Ballot SC-081, maximum public TLS certificate validity falls in stages to **47 days by 2029**, and Let's Encrypt also offers short-lived certificates. **Fully automated renewal is no longer optional.** Any process that needs a human to act every renewal cycle will eventually fail.

### 11.2 Bootstrap and issuance

`certbot` runs on the **host** as root, separate from the container, so the container never holds credentials for the ACME account. It uses the HTTP-01 challenge through the shared, read-only webroot.

```bash
# apt install certbot
# install -d -m 0755 /srv/acme
```

**Bootstrap order:** the HTTPS server block needs a certificate before it exists. For the first start, comment out the `8443` server block in `site.conf` and start the container (§10.6) so port 80 serves ACME challenges. Then:

```bash
# certbot certonly --webroot -w /srv/acme \
      -d example.com -d www.example.com \
      --key-type ecdsa --elliptic-curve secp384r1 \
      --agree-tos -m security@example.com --no-eff-email
```

> **DNS-01 alternative.** DNS-01 lets you obtain certificates with no inbound port 80. It requires API access to your DNS provider, and some registrars restrict DNS API access for small accounts. Moving DNS hosting to a provider with a scoped API token is the usual fix.

### 11.3 Deploying certificates across the user-namespace boundary

In rootless mode, container UID 101 maps to host UID `subuid_start + 100`. The deploy hook must therefore set ownership **in the mapped ID space**, so that the key is readable by NGINX and by nothing else. Create `/etc/letsencrypt/renewal-hooks/deploy/50-web-nginx.sh`:

```sh
#!/bin/sh
# Copy renewed certs into the rootless container's bind mount and reload NGINX.
set -eu

RUNUSER=dockrun
CONTAINER_UID=101
DEST=/home/${RUNUSER}/web/certs

SUBUID_START=$(awk -F: -v u="$RUNUSER" '$1==u {print $2}' /etc/subuid)
SUBGID_START=$(awk -F: -v u="$RUNUSER" '$1==u {print $2}' /etc/subgid)
HOST_UID=$((SUBUID_START + CONTAINER_UID - 1))
HOST_GID=$((SUBGID_START + CONTAINER_UID - 1))

install -d -m 0750 -o "$HOST_UID" -g "$HOST_GID" "$DEST"
install -m 0444 -o "$HOST_UID" -g "$HOST_GID" "$RENEWED_LINEAGE/fullchain.pem" "$DEST/fullchain.pem"
install -m 0400 -o "$HOST_UID" -g "$HOST_GID" "$RENEWED_LINEAGE/privkey.pem"   "$DEST/privkey.pem"

DUID=$(id -u "$RUNUSER")
runuser -u "$RUNUSER" -- env \
    XDG_RUNTIME_DIR="/run/user/$DUID" \
    DOCKER_HOST="unix:///run/user/$DUID/docker.sock" \
    docker exec web-nginx nginx -s reload
```

```bash
# chmod 0750 /etc/letsencrypt/renewal-hooks/deploy/50-web-nginx.sh
# RENEWED_LINEAGE=/etc/letsencrypt/live/example.com \
      /etc/letsencrypt/renewal-hooks/deploy/50-web-nginx.sh     # first run
# systemctl list-timers | grep certbot                         # renewal timer active
# certbot renew --dry-run
```

Then uncomment the `8443` server block and run `docker exec web-nginx nginx -t && docker exec web-nginx nginx -s reload`.

### 11.4 Certificate Authority Authorization (CAA)

Publish a CAA DNS record so that only your chosen CA may issue for your domain. This limits mis-issuance:

```
example.com.  CAA 0 issue "letsencrypt.org"
example.com.  CAA 0 iodef "mailto:security@example.com"
```

### 11.5 Launch

```bash
$ cd ~/web
$ docker compose config          # validates and shows the resolved, pinned image
$ docker compose up -d
$ docker compose ps              # STATUS should become "healthy"
```

---

## 12. Phase VIII — Software Supply-Chain Integrity

The `xz-utils` incident showed that the most dangerous code on a server can be code you installed deliberately. Controls:

1. **Digest pinning** (§10.3): you run exactly the bytes you reviewed.
2. **Vulnerability scanning before deployment.** Use an open-source scanner such as Trivy or Grype:
   ```bash
   $ trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 "$NGINX_IMAGE"
   ```
3. **SBOM generation** (Syft or `docker buildx` attestations) gives you an inventory to search when the *next* library CVE appears ("am I affected?").
4. **Signature and provenance verification.** Docker Content Trust (Notary v1) is being retired. Modern verification uses **Sigstore/cosign** signatures and SLSA provenance attestations where the publisher provides them. Verify when available, and prefer publishers that sign.
5. **Verify your tools too.** Scanners and CI actions are supply-chain targets as well. Install them from signed distribution channels and pin their versions.

### 12.1 Reconciling pinning with freshness

Digest pinning freezes the image, *including its vulnerabilities*. Resolve this with a scheduled **pull → scan → promote** pipeline. Save as `~dockrun/bin/refresh-nginx.sh`:

```bash
#!/usr/bin/env bash
# Resolve the tracked tag to its newest digest; deploy only if the scan passes.
set -euo pipefail
cd "$HOME/web"

TAG="docker.io/nginxinc/nginx-unprivileged:1.28-alpine"
docker pull -q "$TAG" >/dev/null
NEW_REF=$(docker image inspect --format '{{index .RepoDigests 0}}' "$TAG")
NEW_REF="${TAG}@${NEW_REF#*@}"
CUR_REF=$(grep -E '^NGINX_IMAGE=' .env | cut -d= -f2-)

if [[ "$NEW_REF" == "$CUR_REF" ]]; then
    echo "No change: $CUR_REF"; exit 0
fi

trivy image --quiet --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 "$NEW_REF"

cp .env ".env.bak.$(date +%F)"
sed -i "s|^NGINX_IMAGE=.*|NGINX_IMAGE=${NEW_REF}|" .env
docker compose up -d
sleep 20
docker inspect --format '{{.State.Health.Status}}' web-nginx | grep -qx healthy \
    || { echo "Unhealthy — rolling back"; mv ".env.bak.$(date +%F)" .env; docker compose up -d; exit 1; }
git add .env && git commit -qm "nginx image -> ${NEW_REF##*@}" || true
echo "Promoted: $NEW_REF"
```

Schedule it weekly with a systemd **user** timer for `dockrun` (`~/.config/systemd/user/refresh-nginx.{service,timer}`, `OnCalendar=Sun 03:00`, `Persistent=true`).

---

## 13. Phase IX — Logging, Monitoring, and Detection

Prevention eventually fails. The goal then shifts to **reducing dwell time**, the interval between compromise and discovery.

### 13.1 Persistent journal

Debian 12 and later rely on `journald`; `/var/log/auth.log` does not exist unless you install `rsyslog`.

```bash
# mkdir -p /var/log/journal && systemctl restart systemd-journald
# journalctl -u ssh --since today
```

### 13.2 Kernel audit framework

```bash
# apt install auditd audispd-plugins
```

`/etc/audit/rules.d/90-server.rules` (adapted from the CIS Debian and Docker benchmarks):

```
-D
-b 8192
-f 1

# Identity and privilege configuration
-w /etc/passwd   -p wa -k identity
-w /etc/shadow   -p wa -k identity
-w /etc/group    -p wa -k identity
-w /etc/subuid   -p wa -k identity
-w /etc/subgid   -p wa -k identity
-w /etc/sudoers  -p wa -k privilege
-w /etc/sudoers.d/ -p wa -k privilege

# Remote access
-w /etc/ssh/sshd_config   -p wa -k sshd
-w /etc/ssh/sshd_config.d/ -p wa -k sshd

# Container runtime
-w /usr/bin/dockerd         -k docker
-w /usr/bin/containerd      -k docker
-w /usr/bin/runc            -k docker
-w /usr/bin/rootlesskit     -k docker
-w /home/dockrun/.config/docker/ -p wa -k docker
-w /home/dockrun/web/       -p wa -k webconfig

# Firewall and kernel
-w /etc/nftables.conf -p wa -k firewall
-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -k modules

# Privileged command execution by humans
-a always,exit -F arch=b64 -S execve -F euid=0 -F auid>=1000 -F auid!=unset -k root_cmd

# Make rules immutable until reboot (an attacker must reboot to blind auditing)
-e 2
```

```bash
# augenrules --load && auditctl -s
# ausearch -k sshd -ts today
```

### 13.3 File-integrity monitoring

```bash
# apt install aide
# aideinit && mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

Debian runs a daily AIDE check through `dailyaidecheck`. **Store a copy of the baseline database off-host.** An attacker with root can otherwise rewrite it.

### 13.4 Remote log shipping

Logs stored only on a compromised host are evidence the attacker controls. Forward the journal to a separate system (`systemd-journal-upload`, or `rsyslog` over TLS) on a host the web server **cannot log into**.

### 13.5 Runtime detection (optional, advanced)

**Falco** (eBPF-based) detects anomalous container behavior in real time: a shell spawned inside NGINX, writes to binary directories, unexpected outbound connections. Note the inherent tension: Falco itself needs high privilege to observe the kernel, which adds trusted code to the system.

### 13.6 Network-layer abuse response

With SSH closed to the Internet, the remaining public surface is HTTP(S). **CrowdSec**, a collaborative IP-reputation engine with an nftables bouncer, can parse NGINX logs and block scanners. NGINX's `limit_req` (§10.4) already provides baseline protection.

### 13.7 Alert on what matters

At minimum, make sure a human is notified of:

- Any successful SSH login.
- AIDE reports changes outside of patching windows.
- The container becomes `unhealthy` or restarts repeatedly.
- `unattended-upgrades` errors.
- Certificate expiry within 10 days (an external check catches failed renewals).

---

## 14. Phase X — Patch Management and Maintenance Cadence

| Interval | Action | Mechanism |
|---|---|---|
| Daily | Debian security updates installed | `unattended-upgrades` |
| Daily (if needed) | Reboot for kernel/libc updates | `Automatic-Reboot` at 04:30 |
| Daily | Certificate renewal check | `certbot.timer` |
| Daily | File-integrity check | AIDE |
| Weekly | Image refresh: pull → scan → promote | `refresh-nginx.timer` (§12.1) |
| Weekly | Review `debsecan --suite trixie --only-fixed` output | Manual |
| Monthly | Run `lynis audit system` and Docker Bench; compare to baseline | Manual |
| Monthly | Test restore from backup | Manual |
| Per Debian point release | Read release notes; reboot | Manual |
| Continuous | Subscribe to `debian-security-announce` and the NGINX security advisories page | Email |
| Every ~2 years | Major Debian upgrade (13 → 14), ideally by **rebuilding** from this guide | Manual |

**Emergency procedure for a critical, actively exploited CVE** (a new CISA KEV entry affecting your stack): patch out of cycle; if no patch exists, apply the vendor workaround or **take the service offline**. A static site being down for a day costs little; a compromised host costs far more.

---

## 15. Phase XI — Backup, Recovery, and Incident Response

### 15.1 What to back up

| Data | Method | Notes |
|---|---|---|
| `/home/dockrun/web` (minus `certs/`) | Git remote + `restic` | Configuration and content |
| `/etc` | `etckeeper` (Git) + `restic` | Host configuration history |
| `/etc/letsencrypt` | `restic`, **encrypted** | Contains private keys and ACME account key |
| AIDE baseline | Copy off-host after each update | Integrity reference |
| LUKS header | `cryptsetup luksHeaderBackup`, stored offline | Header corruption equals total data loss |

Follow **3-2-1-1**: three copies, two media types, one off-site, **one immutable or offline**, so ransomware on the host cannot delete its own backups.

### 15.2 Incident response principles

If compromise is suspected:

1. **Contain**: cut network access at the edge firewall. Do not power off yet, because volatile memory holds evidence.
2. **Preserve**: export logs (already shipped off-host, §13.4), `docker inspect`, `docker diff web-nginx`, the process list, and network connections.
3. **Eradicate by rebuilding.** **Do not attempt to "clean" a compromised Linux host.** Rootkits, modified binaries, and hidden persistence are impractical to rule out. Reinstall from verified media using this guide and your version-controlled configuration.
4. **Rotate all secrets**: SSH keys, the TLS key (reissue the certificate), ACME account, and any API tokens.
5. **Review**: identify the initial vector and add a control that would have prevented or detected it.

---

## 16. Verification and Validation

A control that has not been tested should be treated as missing. Run these after deployment and after significant changes.

### 16.1 Boot and kernel

```bash
$ mokutil --sb-state                        # SecureBoot enabled
$ cat /sys/kernel/security/lockdown          # [integrity] selected
$ sudo aa-status | head
$ sysctl kernel.kptr_restrict kernel.unprivileged_bpf_disabled
```

### 16.2 Network exposure (from an external host)

```bash
$ nmap -Pn -sS -p- --reason <public-ip>       # expect only 80/tcp and 443/tcp open
$ nmap -Pn -6 -sS -p- <public-ipv6>           # IPv6 is often forgotten
$ sudo ss -tulpn                              # on host: compare listeners to expectations
```

### 16.3 SSH

```bash
$ ssh-audit <server>                          # from the management LAN
$ ssh -o PubkeyAuthentication=no admin@server # must be refused
```

### 16.4 TLS and HTTP headers

```bash
$ testssl.sh --severity MEDIUM https://example.com
$ curl -sSI https://example.com | grep -iE 'strict-transport|content-security|x-content-type|server:'
$ curl -sS -X POST -o /dev/null -w '%{http_code}\n' https://example.com/   # expect 405
$ curl -sS -o /dev/null -w '%{http_code}\n' https://example.com/.git/config # expect 404
```

Also check with the Mozilla HTTP Observatory and the Qualys SSL Labs server test.

### 16.5 Container posture

```bash
$ docker exec web-nginx id                              # uid=101
$ docker exec web-nginx sh -c 'touch /usr/share/nginx/html/x'   # must fail: Read-only
$ docker exec web-nginx sh -c 'cp /bin/busybox /tmp/b && /tmp/b' # must fail: noexec
$ docker inspect web-nginx --format \
  'ro={{.HostConfig.ReadonlyRootfs}} caps={{.HostConfig.CapDrop}} sec={{.HostConfig.SecurityOpt}} pids={{.HostConfig.PidsLimit}} mem={{.HostConfig.Memory}}'
$ grep CapEff /proc/$(docker inspect -f '{{.State.Pid}}' web-nginx)/status  # 0000000000000000
$ ps -o user,pid,cmd -C nginx                            # host view: subordinate UIDs, not root
```

### 16.6 Benchmarks

```bash
# apt install lynis && lynis audit system          # record the hardening index as a baseline
$ git clone https://github.com/docker/docker-bench-security && cd docker-bench-security
$ sh docker-bench-security.sh                     # some checks assume rootful mode; interpret accordingly
$ systemd-analyze security                        # exposure score per system service
```

---

## 17. Discussion: Limitations and Open Trade-offs

1. **Containers share the host kernel.** No container configuration defends against a kernel exploit reachable through allowed syscalls. Stronger isolation needs a different boundary: **gVisor** (`runsc`, a userspace kernel) or **Kata Containers** (lightweight VMs). Both have performance and compatibility costs, and Kata needs hardware virtualization.
2. **No AppArmor in rootless mode.** This design gives up per-container MAC in exchange for an unprivileged daemon (§9.2). Seccomp, dropped capabilities, and the user namespace partly compensate. Rootless Podman on an SELinux-based distribution provides both, at the cost of leaving Debian.
3. **User-namespace attack surface** remains the main residual risk of Model A. The mitigations are fast kernel patching and module blacklisting. Some distributions restrict unprivileged user namespaces through AppArmor policy, an approach Debian does not use by default.
4. **Trust in upstream** (Debian, Docker Inc., the NGINX image maintainers, Let's Encrypt) is reduced by verification but not removed. Reproducible builds, which Debian actively pursues, are the long-term answer.
5. **Physical security** of a home-hosted server is weak. TPM-based auto-unlock protects a stolen disk, not a stolen machine.
6. **Availability.** A single host, nightly reboots, and a residential uplink give modest uptime. For higher availability, put a CDN in front, or keep a static mirror on a managed static-hosting platform as a failover.
7. **Operator error** is still the most likely cause of failure. Version-controlled configuration, the verification suite in §16, and the checklist in Appendix A are the procedural controls against it.

---

## 18. Conclusion

Hosting a website on Linux in 2026 is less about any single configuration and more about **systems thinking under constant vulnerability pressure**. The architecture here arranges independent barriers so that no single vulnerability is fatal:

- an unprivileged, capability-less, read-only NGINX
- inside a user-namespaced container
- managed by an unprivileged daemon
- on a minimal, automatically patched, Secure-Boot-locked host
- reachable only on ports 80 and 443
- administered only from trusted networks with hardware-backed keys

It also accepts that barriers fail, and adds detection, off-host evidence, and a rebuild-not-repair recovery model. What makes this sustainable is that the **maintenance is automated**: in the current threat environment, a server that depends on a person remembering to patch it will eventually go unpatched.

---

## 19. References

**Standards and frameworks**

1. NIST SP 800-190, *Application Container Security Guide*, 2017.
2. NIST SP 800-123, *Guide to General Server Security*, 2008.
3. NIST SP 800-53 Rev. 5, *Security and Privacy Controls for Information Systems and Organizations*.
4. NIST FIPS 203, *Module-Lattice-Based Key-Encapsulation Mechanism Standard (ML-KEM)*, 2024.
5. Center for Internet Security, *CIS Debian Linux Benchmark* and *CIS Docker Benchmark* (current versions).
6. MITRE ATT&CK® for Enterprise — techniques T1068, T1110, T1190, T1195, T1496, T1505.003, T1611.
7. CA/Browser Forum, Ballot SC-081: *Introduce Schedule of Reducing Validity and Data Reuse Periods*.

**RFCs**

8. RFC 8446 — The Transport Layer Security (TLS) Protocol Version 1.3.
9. RFC 6797 — HTTP Strict Transport Security (HSTS).
10. RFC 8555 — Automatic Certificate Management Environment (ACME).
11. RFC 8659 — DNS Certification Authority Authorization (CAA).
12. RFC 8915 — Network Time Security for the Network Time Protocol.
13. RFC 9116 — A File Format to Aid in Security Vulnerability Disclosure (security.txt).

**Vendor and project documentation**

14. Debian Project, *Securing Debian Manual* and *Debian 13 Release Notes* — debian.org/doc.
15. Debian Project, *Verifying authenticity of Debian images* — debian.org/CD/verify.
16. Docker Inc., *Install Docker Engine on Debian*; *Run the Docker daemon as a non-root user (Rootless mode)*; *Docker security*; *Packet filtering and firewalls* — docs.docker.com.
17. F5/NGINX, *nginx documentation* and *nginx security advisories* — nginx.org.
18. Mozilla, *Server Side TLS* guidelines and SSL Configuration Generator — ssl-config.mozilla.org.
19. OWASP, *Docker Security Cheat Sheet*; *HTTP Headers Cheat Sheet*.
20. OpenSSH, *Release Notes* (9.8: PerSourcePenalties; 9.9/10.0: ML-KEM hybrid key exchange) — openssh.com.
21. Let's Encrypt, announcements on ending OCSP support and on short-lived certificates — letsencrypt.org.

**Vulnerability records** (NVD / CISA KEV)

22. CVE-2024-3094 (xz-utils backdoor).
23. CVE-2024-6387 (OpenSSH "regreSSHion").
24. CVE-2024-1086 (Linux nf_tables use-after-free).
25. CVE-2024-21626 (runc "Leaky Vessels").
26. CVE-2025-31133, CVE-2025-52565, CVE-2025-52881 (runc container escapes).
27. CVE-2025-32463 (sudo chroot privilege escalation).
28. CISA, *Known Exploited Vulnerabilities Catalog* — cisa.gov/known-exploited-vulnerabilities-catalog.

---

## Appendix A — Deployment Checklist

**Installation**
- [ ] ISO signature and checksum verified against an independently obtained fingerprint
- [ ] UEFI Secure Boot enabled; firmware password set
- [ ] LUKS2 (argon2id) full-disk encryption; header backed up offline
- [ ] Separate `/var`, `/var/log`, `/var/log/audit`, `/home`
- [ ] Only "SSH server" and "standard utilities" installed

**Operating system**
- [ ] Root account locked; admin in `sudo` and `sshusers`
- [ ] `unattended-upgrades` with automatic reboot configured
- [ ] `/etc/sysctl.d/90-hardening.conf` applied
- [ ] Unused kernel modules blacklisted; initramfs rebuilt
- [ ] `/tmp`, `/dev/shm` mounted `noexec,nosuid,nodev`
- [ ] AppArmor enforcing; kernel lockdown = integrity
- [ ] Chrony with NTS

**Remote access**
- [ ] FIDO2 (`ed25519-sk`) or Ed25519 key-only authentication
- [ ] `PermitRootLogin no`, `PasswordAuthentication no`, `AllowGroups sshusers`
- [ ] PQ-hybrid KEX configured; `ssh-audit` clean
- [ ] SSH reachable **only** from management LAN / VPN

**Network**
- [ ] nftables default-deny inbound; 80/443 public; 22 allow-listed
- [ ] IPv6 ruleset verified (external IPv6 scan)
- [ ] Edge forwards only 80/443; host isolated from internal LAN

**Container runtime**
- [ ] Rootful Docker daemon disabled; socket removed
- [ ] Rootless Docker under `dockrun` (no sudo, no SSH); linger enabled
- [ ] cgroup v2 controllers delegated
- [ ] `slirp4netns` port driver (client IPs preserved)
- [ ] `daemon.json`: `icc=false`, `no-new-privileges`, `live-restore`, bounded logs

**Workload**
- [ ] `nginx-unprivileged` image pinned by **digest**
- [ ] `cap_drop: ALL`, `no-new-privileges`, `read_only`, `noexec` tmpfs
- [ ] All mounts `:ro`
- [ ] CPU, memory, and PID limits set
- [ ] Healthcheck reports `healthy`

**TLS**
- [ ] TLS 1.2/1.3 only; Mozilla intermediate ciphers; session tickets off
- [ ] HSTS, CSP, and companion headers present
- [ ] Certbot timer active; `renew --dry-run` passes; deploy hook reloads NGINX
- [ ] CAA record published

**Supply chain and operations**
- [ ] Weekly pull → scan → promote timer installed and tested (including rollback)
- [ ] auditd rules loaded and immutable (`-e 2`)
- [ ] AIDE baseline created and copied off-host
- [ ] Logs shipped to a separate host
- [ ] Alerts for SSH logins, AIDE changes, unhealthy container, certificate expiry
- [ ] Backups follow 3-2-1-1; restore tested
- [ ] Lynis and Docker Bench baselines recorded
- [ ] All configuration committed to version control

---

*This document describes defensive configuration for systems you own or administer. Verify all version-specific details against current upstream documentation before production use.*
