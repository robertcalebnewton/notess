# OpenBSD Firewall: Home Lab Reference Guide

A beginner's guide to building an OpenBSD router/firewall that sits between your modem and your network. It also covers safely hosting your Debian + nginx (container) website.

> **How to use this guide:** Read sections 1–3 before buying or installing anything. Sections 4–9 are the build steps, in order. Sections 10–13 are reference material to come back to.

---

## Table of Contents

1. [Core concepts you need to know](#1-core-concepts-you-need-to-know)
2. [Your network design](#2-your-network-design)
3. [Before you start: checklist](#3-before-you-start-checklist)
4. [Install OpenBSD](#4-install-openbsd)
5. [Configure the network interfaces](#5-configure-the-network-interfaces)
6. [DHCP and DNS for your home network](#6-dhcp-and-dns-for-your-home-network)
7. [The firewall rules (pf.conf)](#7-the-firewall-rules-pfconf)
8. [Set up the Deco mesh](#8-set-up-the-deco-mesh)
9. [Set up the Debian web server](#9-set-up-the-debian-web-server)
10. [Testing checklist](#10-testing-checklist)
11. [Security hardening checklist](#11-security-hardening-checklist)
12. [Maintenance and monitoring](#12-maintenance-and-monitoring)
13. [Command cheat sheet](#13-command-cheat-sheet)
14. [Troubleshooting](#14-troubleshooting)
15. [Glossary](#15-glossary)
16. [Further reading](#16-further-reading)

---

## 1. Core concepts you need to know

### What a firewall does
A firewall inspects every packet that passes through it and decides, based on your rules, whether to **pass** or **block** it. Your OpenBSD box also acts as a **router**: it moves traffic between networks (the internet and your home networks).

### Default deny
The most important rule in firewall design: **block everything, then allow only what you need.** Never do the reverse (allow everything, block known-bad stuff). Your `pf.conf` will start with `block all`.

### Stateful filtering
PF remembers connections ("states"). When your laptop opens a connection to a website, PF records it and automatically allows the reply traffic back in. You only write rules for who is allowed to **start** a connection.

### NAT (Network Address Translation)
Your ISP gives you **one public IP address**. All your devices use **private addresses** (like `10.0.10.x`). NAT rewrites outgoing traffic so it appears to come from your public IP, and maps replies back to the right device.

### Port forwarding (rdr-to)
Because of NAT, nobody on the internet can reach your private devices directly. **Port forwarding** says "traffic arriving at my public IP on port 443, send it to the web server." In PF this is `rdr-to`.

### DMZ (Demilitarized Zone)
A separate network for anything exposed to the internet, like your web server. If the web server is ever hacked, the attacker is stuck in the DMZ and **cannot reach your home devices** (phones, laptops, NAS, etc.). This is the single biggest safety win in this design.

### Subnets and CIDR
`10.0.10.0/24` means "addresses `10.0.10.0` through `10.0.10.255`." The `/24` means the first 24 bits (the first three numbers) are fixed. Each network (LAN, DMZ) gets its own subnet.

### PF rule order: last match wins
**This trips up every beginner.** PF reads every rule top to bottom, and the **last rule that matches** a packet decides its fate, unless a rule has the `quick` keyword. A `quick` rule takes effect immediately and stops evaluation.

```
block all                 # matches everything...
pass in on em1            # ...but this matches later, so em1 traffic passes
```

---

## 2. Your network design

### Physical layout (recommended: 3 network ports on the firewall)

```
     Internet
         │
   ┌─────┴─────┐
   │   Modem   │   (bridge mode, if it's a modem/router combo)
   └─────┬─────┘
         │
   em0 (WAN) - gets public IP from ISP via DHCP
   ┌─────┴──────────────────────────┐
   │       OpenBSD Firewall         │
   │  PF · NAT · DHCP · DNS (unbound)│
   └──┬──────────────────────────┬──┘
      │                          │
  em1 (LAN)                  em2 (DMZ)
  10.0.10.1/24               10.0.20.1/24
      │                          │
┌─────┴──────┐            ┌──────┴────────┐
│ Deco Mesh  │            │ Debian Server │
│ (AP mode)  │            │ 10.0.20.10    │
└─────┬──────┘            │ nginx (docker)│
      │                   └───────────────┘
 Phones, laptops,
 TVs, etc.
 10.0.10.100-199
```

### Address plan

| Network | Interface | Subnet | Firewall IP | Purpose |
|---|---|---|---|---|
| WAN | `em0` | From ISP | DHCP | Internet |
| LAN | `em1` | `10.0.10.0/24` | `10.0.10.1` | Trusted home devices (via Deco) |
| DMZ | `em2` | `10.0.20.0/24` | `10.0.20.1` | Web server only |

> Interface names depend on your network card's driver: `em` and `igc` for Intel, `re` for Realtek, etc. Run `ifconfig` after installing to see yours, then substitute them everywhere in this guide.

### Traffic policy (what's allowed)

| From → To | Internet | LAN | DMZ | Firewall itself |
|---|---|---|---|---|
| **Internet** | — | ❌ | ✅ ports 80/443 only | ❌ |
| **LAN** | ✅ | ✅ | ✅ (to manage server) | ✅ (SSH, DNS, DHCP) |
| **DMZ** | ✅ (updates) | ❌ | — | DNS only |

### Only have 2 network ports?
Options, from best to worst:
1. **Add a NIC.** A cheap Intel PCIe or USB-to-Ethernet adapter can work. Check the hardware against OpenBSD's support list first.
2. **VLANs.** Use a managed switch and OpenBSD's `vlan(4)` interfaces to make LAN and DMZ separate networks on one port. This is more advanced.
3. **Put the server on the LAN.** It works, but you lose the DMZ isolation. If the server is compromised, the attacker can reach your home devices.

---

## 3. Before you start: checklist

### Hardware for the firewall
- [ ] A small PC or mini PC with **2–3+ Ethernet ports**. **Intel NICs strongly preferred** (i210/i211/i225/i226 work well).
- [ ] 2–4 GB RAM is plenty. 16 GB+ SSD.
- [ ] Doesn't need Wi-Fi; your Deco handles that.
- [ ] A **monitor and keyboard** for the initial setup. Don't rely on SSH until it's working; a bad firewall rule can lock you out.
- [ ] A USB stick (1 GB+) for the installer.

### Check your ISP situation
- [ ] **Is your modem a modem/router combo?** If so, put it in **bridge mode** (or "IP passthrough") so the firewall gets the public IP directly. Without this you'll have **double NAT**, and port forwarding gets messy.
- [ ] **Are you behind CGNAT?** Some ISPs share one public IP among many customers. Check: after setup, compare the WAN IP on the firewall (`ifconfig em0`) to what a site like `ifconfig.me` reports. If they differ, or the WAN IP starts with `100.64`–`100.127`, you're behind CGNAT and **can't host directly**. Use a tunnel service (for example Cloudflare Tunnel) instead.
- [ ] **Does your ISP allow hosting?** Some residential plans block inbound ports 80/443 or forbid servers in their terms of service.
- [ ] **Your public IP probably changes.** You'll need **Dynamic DNS** so your domain always points to your current IP. Many DNS providers offer an API; tools like `ddclient` can update it automatically.

### Have a rollback plan
- [ ] Keep your current setup (modem → Deco as router) intact until the new firewall is fully tested. If something breaks, plug things back the old way.

---

## 4. Install OpenBSD

1. Download the latest `install*.img` for **amd64** from <https://www.openbsd.org> (check the latest release on the homepage).
2. Write it to a USB stick (e.g., with balenaEtcher, or `dd` on Linux/macOS).
3. Boot the firewall from USB and choose **(I)nstall**.

### Installer answers (the important ones)

| Prompt | Suggested answer |
|---|---|
| Hostname | `fw` (or anything) |
| Network interface to configure | Configure `em0` → `autoconf` (DHCP). Configure `em1` → `10.0.10.1`, netmask `255.255.255.0`. Configure `em2` → `10.0.20.1`, netmask `255.255.255.0`. |
| Root password | Strong and unique |
| Start sshd by default? | `yes` (PF will restrict it to the LAN) |
| Do you expect to run X Window System? | `no` |
| Setup a user? | `yes`, and use this account day to day |
| Allow root ssh login? | `no` |
| Sets to install | Type `-x* -game*` to skip graphics and games. It's a server; less software means less attack surface. |

4. Reboot, remove the USB, log in.

### First things after install
```sh
# Apply security patches
syspatch

# Allow your user to run admin commands (OpenBSD uses doas, not sudo)
echo "permit persist :wheel" > /etc/doas.conf
# (your user is in the wheel group if you created it during install)
```

---

## 5. Configure the network interfaces

The installer usually creates these files, but here's what they should contain. OpenBSD stores each interface's config in `/etc/hostname.<interface>`.

```sh
# /etc/hostname.em0  (WAN - get IP from ISP)
inet autoconf
```

```sh
# /etc/hostname.em1  (LAN)
inet 10.0.10.1 255.255.255.0
```

```sh
# /etc/hostname.em2  (DMZ)
inet 10.0.20.1 255.255.255.0
```

### Turn on routing (IP forwarding)
By default OpenBSD **won't** route traffic between interfaces. Enable it:

```sh
echo 'net.inet.ip.forwarding=1' >> /etc/sysctl.conf   # survives reboot
sysctl net.inet.ip.forwarding=1                       # apply now
```

Apply interface changes without rebooting:
```sh
sh /etc/netstart
```

> **IPv6:** This guide is IPv4-only to keep things simple. The rules below block IPv6 by default. Add IPv6 later, once everything else works.

---

## 6. DHCP and DNS for your home network

### DHCP server (hands out IPs to LAN devices)

`/etc/dhcpd.conf`:
```
option domain-name-servers 10.0.10.1;

subnet 10.0.10.0 255.255.255.0 {
    option routers 10.0.10.1;
    range 10.0.10.100 10.0.10.199;

    # Example: always give a device the same IP (optional)
    # host printer {
    #     hardware ethernet aa:bb:cc:dd:ee:ff;
    #     fixed-address 10.0.10.20;
    # }
}
```

Enable it on the **LAN only**. The DMZ server uses a static IP, so it doesn't need DHCP:
```sh
rcctl enable dhcpd
rcctl set dhcpd flags em1
rcctl start dhcpd
```

### DNS resolver (unbound, included with OpenBSD)

Edit `/var/unbound/etc/unbound.conf`. Keep the existing defaults and make sure the `server:` section has:

```
server:
    interface: 127.0.0.1
    interface: 10.0.10.1
    interface: 10.0.20.1

    access-control: 0.0.0.0/0 refuse
    access-control: 127.0.0.0/8 allow
    access-control: 10.0.10.0/24 allow
    access-control: 10.0.20.0/24 allow

    hide-identity: yes
    hide-version: yes

    # Split DNS: when you're at home, your domain resolves to the
    # server's private IP instead of your public IP. Replace with your domain.
    local-zone: "example.com." transparent
    local-data: "example.com. IN A 10.0.20.10"
    local-data: "www.example.com. IN A 10.0.20.10"
```

> **Why split DNS?** Without it, visiting your own site from home goes out to your public IP and has to loop back in ("hairpin NAT"), which often doesn't work. Split DNS sends home devices straight to the server.

```sh
unbound-checkconf          # check for typos
rcctl enable unbound
rcctl start unbound
```

---

## 7. The firewall rules (pf.conf)

This is the heart of the firewall. It's adapted from OpenBSD's official "Building a Router" FAQ, with a DMZ added.

`/etc/pf.conf`:
```
# ================================================================
#  /etc/pf.conf - home lab firewall
#  Reminder: without "quick", the LAST matching rule wins.
# ================================================================

# ---------- 1. Macros (variables) ----------
lan        = "em1"                 # Deco mesh / home devices
dmz        = "em2"                 # Debian web server
web_server = "10.0.20.10"
web_ports  = "{ http, https }"     # 80 and 443
# Note: "egress" is a built-in group = the interface with the
# default route (em0, your WAN). No macro needed for it.

# ---------- 2. Tables ----------
# Addresses that should never appear as a source on the internet side
table <martians> const { 0.0.0.0/8 10.0.0.0/8 127.0.0.0/8 169.254.0.0/16 \
                         172.16.0.0/12 192.0.0.0/24 192.0.2.0/24 224.0.0.0/3 \
                         192.168.0.0/16 198.18.0.0/15 198.51.100.0/24 \
                         203.0.113.0/24 }

# Filled automatically with IPs that hammer your web server (see section 11)
table <abusers> persist

# ---------- 3. Options ----------
set block-policy drop         # silently drop blocked packets
set loginterface egress       # collect stats on the WAN interface
set skip on lo0               # don't filter loopback

# ---------- 4. Normalize packets ----------
match in all scrub (no-df random-id max-mss 1440)

# ---------- 5. NAT: LAN + DMZ share the public IP ----------
match out on egress inet from !(egress:network) to any nat-to (egress:0)

# ---------- 6. Default deny + anti-spoofing ----------
antispoof quick for { egress $lan $dmz }
block in  quick on egress from <martians> to any
block return out quick on egress from any to <martians>
block in  quick on egress from <abusers>
block log all                 # everything not allowed below is blocked + logged

# ---------- 7. The firewall may send traffic out ----------
pass out quick inet

# ---------- 8. LAN: trusted ----------
# Home devices can reach the internet, the DMZ, and the firewall.
pass in on $lan inet

# ---------- 9. DMZ: internet only ----------
# The web server can go out to the internet (for updates, certs)...
pass in on $dmz inet
# ...but can NOT reach your home LAN or the firewall itself...
block log in on $dmz inet to { $lan:network self }
# ...except to use the firewall for DNS lookups and ping.
pass in on $dmz inet proto { tcp udp } to ($dmz) port domain
pass in on $dmz inet proto icmp to ($dmz)

# ---------- 10. Port forward: internet -> web server ----------
# Rate limit: an IP that opens >100 connections at once, or >30
# in 5 seconds, is added to <abusers> and blocked.
pass in on egress inet proto tcp to (egress) port $web_ports \
    rdr-to $web_server \
    keep state (max-src-conn 100, max-src-conn-rate 30/5, \
                overload <abusers> flush global)
```

### Load the rules safely

```sh
pfctl -nf /etc/pf.conf     # -n = check syntax only, don't load
pfctl -f /etc/pf.conf      # load the rules
pfctl -sr                  # show the loaded rules
```

> ⚠️ **Always run `pfctl -nf` first**, and do your first loads **at the console**, not over SSH. A mistake can cut off your own connection.

### Rules explained in plain English

| Rule | What it means |
|---|---|
| `antispoof` | Blocks packets claiming to come from a network that's on a different port (a common attack trick). |
| `<martians>` | Private and reserved addresses should never arrive from the internet; drop them. |
| `block log all` | The default deny. Anything not explicitly allowed is dropped and logged. |
| `pass out quick inet` | Once traffic is allowed in, it can leave out the other side. |
| `pass in on $lan` | Your home devices are trusted. |
| DMZ rules | The web server is treated as **untrusted**: internet yes, your home no. |
| `rdr-to` | Port forwarding: web traffic arriving at your public IP goes to the server. |

### Notice what's NOT in the rules
- No rule allows the internet to reach the firewall itself. **SSH is only reachable from the LAN.**
- No rule allows the internet to reach any LAN device.
- No ports are open besides 80/443 → web server.

> **If your modem can't do bridge mode** and gives the firewall a private IP like `192.168.0.x`, remove `192.168.0.0/16` from `<martians>`. Otherwise you'll block traffic from your own modem.

---

## 8. Set up the Deco mesh

The firewall now does routing, DHCP, DNS, and firewalling, so the Deco should **only** provide Wi-Fi.

1. In the **Deco app**, go to the advanced settings and change **Operation Mode** to **Access Point** mode.
2. Plug the firewall's **LAN port (`em1`)** into the main Deco unit's Ethernet port.
3. The Deco and all wireless devices will now get `10.0.10.x` addresses from the firewall.

**Why Access Point mode?** If the Deco stays in Router mode, you get double NAT. Your devices end up behind two routers, the firewall sees them all as a single IP, and troubleshooting gets confusing.

**Trade-off:** In AP mode, some Deco features (like its built-in parental controls or security features) may be disabled, since the Deco is no longer routing traffic.

Also:
- [ ] Change the Deco admin password.
- [ ] Update the Deco firmware.
- [ ] Use WPA2/WPA3 with a strong Wi-Fi password.

---

## 9. Set up the Debian web server

### Give it a static IP
Plug the server into the firewall's **DMZ port (`em2`)**. Edit `/etc/network/interfaces` (replace `enp1s0` with your interface name from `ip a`):

```
auto enp1s0
iface enp1s0 inet static
    address 10.0.20.10/24
    gateway 10.0.20.1
```

`/etc/resolv.conf`:
```
nameserver 10.0.20.1
```

Then: `sudo systemctl restart networking`, or reboot.

> If your Debian uses NetworkManager or systemd-networkd instead, set the same values there.

### nginx in a container
Publish ports 80 and 443 from the container, e.g.:
```sh
docker run -d --name web -p 80:80 -p 443:443 ... nginx
```

> ⚠️ **Docker gotcha:** Docker edits the server's iptables directly and **bypasses host firewalls like `ufw`**. Don't rely on `ufw` to protect container ports. Your **OpenBSD firewall + DMZ** is the real protection, which is one reason this design matters.

### HTTPS certificates
- Use **Let's Encrypt** (certbot, acme.sh, or a reverse proxy like Caddy/Traefik that handles it automatically).
- The HTTP challenge needs port **80** reachable from the internet (already forwarded). If your ISP blocks port 80, use the **DNS challenge** instead.

### Harden the server
- [ ] SSH key login only. In `/etc/ssh/sshd_config`: `PasswordAuthentication no`, `PermitRootLogin no`.
- [ ] Automatic security updates: `sudo apt install unattended-upgrades`.
- [ ] Keep container images updated (`docker pull` regularly).
- [ ] Run the container as a non-root user where possible.
- [ ] You'll manage the server from your LAN (e.g., `ssh you@10.0.20.10`). The firewall allows LAN → DMZ but not the reverse.

---

## 10. Testing checklist

Test in this order. If one step fails, fix it before moving on.

### From a laptop on the LAN (via Deco Wi-Fi or Ethernet)
- [ ] Got a `10.0.10.x` address? (`ipconfig` / `ifconfig`)
- [ ] `ping 10.0.10.1` (firewall) works
- [ ] `ping 1.1.1.1` works (routing + NAT)
- [ ] `nslookup openbsd.org` works (DNS)
- [ ] Websites load in a browser
- [ ] `ssh you@10.0.20.10` works (LAN → DMZ allowed)
- [ ] Your domain resolves to `10.0.20.10` (split DNS)

### From the Debian server (DMZ)
- [ ] `ping 1.1.1.1` works (internet allowed)
- [ ] `sudo apt update` works
- [ ] `ping 10.0.10.1` **fails** (DMZ → LAN blocked)
- [ ] `ping` any LAN device **fails**
- [ ] `ssh 10.0.20.1` **fails** (DMZ → firewall blocked)

### From outside (phone on cellular data, Wi-Fi OFF)
- [ ] `https://yourdomain.com` loads
- [ ] Nothing else is reachable. Scan your public IP from outside your network (e.g., `nmap` from a cloud server, or an online port scanner). **Only 80 and 443 should be open.**

### On the firewall
- [ ] `pfctl -si` shows counters increasing
- [ ] `tcpdump -n -e -ttt -i pflog0` shows blocked traffic (internet noise appears fast)

---

## 11. Security hardening checklist

### Firewall
- [ ] SSH key-only: in `/etc/ssh/sshd_config`, set `PasswordAuthentication no` and `PermitRootLogin no`
- [ ] Optionally make SSH listen only on the LAN: `ListenAddress 10.0.10.1`
- [ ] Use `doas` from your user account, not root logins
- [ ] Nothing listening on the WAN: check `netstat -an -f inet | grep LISTEN`
- [ ] Install as few packages as possible on the firewall
- [ ] Run `syspatch` whenever a new errata/patch is announced
- [ ] Subscribe to the `announce@openbsd.org` mailing list for security notices
- [ ] Back up `/etc/pf.conf`, `/etc/dhcpd.conf`, `/etc/hostname.*`, `/var/unbound/etc/unbound.conf`, `/etc/sysctl.conf`
- [ ] Clear old abusers periodically. Add to root's crontab (`crontab -e`):
  ```
  0 * * * * /sbin/pfctl -t abusers -T expire 86400
  ```
  (removes entries older than 24 hours, checked hourly)

### Network
- [ ] Web server lives in the DMZ, never on the LAN
- [ ] Deco in AP mode, firmware updated, admin password changed
- [ ] Modem in bridge mode (if possible), admin password changed

### Web server
- [ ] Key-only SSH, unattended upgrades, updated containers
- [ ] HTTPS everywhere (redirect 80 → 443)
- [ ] Don't expose admin panels (database UIs, dashboards) through the port forward

---

## 12. Maintenance and monitoring

### Regular schedule

| How often | Task | Command |
|---|---|---|
| When announced (check weekly) | Security patches | `syspatch` |
| Weekly | Update packages (if any) | `pkg_add -u` |
| Every ~6 months | Upgrade to new OpenBSD release | `sysupgrade` |
| Monthly | Review blocked traffic and abusers | `pfctl -t abusers -T show` |
| After any change | Back up config files | copy `/etc` files somewhere safe |

> **Important:** Each OpenBSD release is supported for about **one year** (two releases). You must upgrade at least yearly to keep getting security patches. `sysupgrade` makes this easy.

### Live monitoring (on the firewall)
- `systat`: press keys to switch views: `pf`, `states`, `rules`, `queues`, `ifstat`
- `pftop`: live connection view (install with `pkg_add pftop`)
- `tcpdump -n -e -ttt -i pflog0`: watch blocked traffic in real time

### Dashboard (later)
Run **Prometheus + Grafana on another machine** (not the firewall). Install `node_exporter` plus a PF exporter on the firewall. There are ready-made Grafana dashboards for OpenBSD PF.

---

## 13. Command cheat sheet

### PF (firewall)
| Command | What it does |
|---|---|
| `pfctl -nf /etc/pf.conf` | Check rules for errors (doesn't load) |
| `pfctl -f /etc/pf.conf` | Load/reload rules |
| `pfctl -e` / `pfctl -d` | Enable / disable PF (**disable = no firewall!**) |
| `pfctl -sr` | Show loaded rules |
| `pfctl -vvsr` | Show rules with hit counters |
| `pfctl -ss` | Show current connections (states) |
| `pfctl -si` | Show overall stats |
| `pfctl -t abusers -T show` | List blocked abusers |
| `pfctl -t abusers -T delete 1.2.3.4` | Unblock an IP |
| `tcpdump -n -e -ttt -i pflog0` | Watch logged/blocked packets live |

### System and services
| Command | What it does |
|---|---|
| `ifconfig` | Show interfaces and IPs |
| `sh /etc/netstart` | Re-apply interface configs |
| `rcctl ls on` | List enabled services |
| `rcctl restart <service>` | Restart a service (e.g., `unbound`, `dhcpd`) |
| `rcctl check <service>` | Is it running? |
| `syspatch` | Install security patches |
| `sysupgrade` | Upgrade to the next release |
| `pkg_add <name>` / `pkg_add -u` | Install / update packages |
| `tail -f /var/log/messages` | Watch system log |
| `cat /var/db/dhcpd.leases` | See devices that got DHCP addresses |
| `man <topic>` | OpenBSD's man pages are excellent. Try `man pf.conf` |

---

## 14. Troubleshooting

| Symptom | Likely cause | Check / fix |
|---|---|---|
| LAN devices get no IP | dhcpd not running, or wrong interface | `rcctl check dhcpd`; `rcctl get dhcpd flags` should show `em1` |
| Devices have an IP but no internet | Forwarding off, or NAT rule wrong | `sysctl net.inet.ip.forwarding` should be `1`; `pfctl -sr` |
| Can ping `1.1.1.1` but websites don't load | DNS problem | `rcctl check unbound`; `unbound-checkconf` |
| Firewall WAN has no IP | Modem not bridged, or needs a reboot | Power-cycle the modem *after* connecting the firewall; `ifconfig em0` |
| Website works from home but not outside | Port forward, CGNAT, or ISP blocking | `pfctl -sr \| grep rdr`; check for CGNAT (section 3); check ISP ToS |
| Website works outside but not at home | Missing split DNS | Add `local-data` entries in unbound (section 6) |
| Locked out after a rule change | Bad rule | At the console: `pfctl -d`, fix `pf.conf`, `pfctl -nf`, then `pfctl -f` and `pfctl -e` |
| Something is blocked and you don't know why | Default deny is doing its job | `tcpdump -n -e -ttt -i pflog0` shows which rule blocked it |

---

## 15. Glossary

| Term                         | Meaning                                                          |
| ---------------------------- | ---------------------------------------------------------------- |
| **WAN**                      | Wide Area Network: the internet side                             |
| **LAN**                      | Local Area Network: your trusted home network                    |
| **DMZ**                      | Isolated network for internet-facing servers                     |
| **NAT**                      | Translating private IPs to your public IP                        |
| **Port forwarding / rdr-to** | Sending inbound traffic on a port to an internal machine         |
| **Stateful**                 | Firewall tracks connections and allows replies automatically     |
| **Egress**                   | PF's name for the interface with the default route (your WAN)    |
| **Martians**                 | Addresses that should never appear from the internet             |
| **DHCP**                     | Automatically hands out IP addresses to devices                  |
| **DNS**                      | Translates names (example.com) into IP addresses                 |
| **Split DNS**                | Different DNS answers inside vs. outside your network            |
| **Bridge mode**              | Modem passes the public IP straight through instead of routing   |
| **Double NAT**               | Two routers doing NAT in a row; causes port-forwarding headaches |
| **CGNAT**                    | ISP shares one public IP among customers; blocks self-hosting    |
| **Access Point mode**        | Deco provides only Wi-Fi, no routing                             |
| **CIDR (/24)**               | Shorthand for subnet size; `/24` = 256 addresses                 |
| **syspatch**                 | OpenBSD's security-patch tool                                    |
| **doas**                     | OpenBSD's simpler alternative to `sudo`                          |

---

## 16. Further reading

- **OpenBSD FAQ: Building a Router**: <https://www.openbsd.org/faq/pf/example1.html> (the official version of this setup)
- **PF User's Guide**: <https://www.openbsd.org/faq/pf/>
- **OpenBSD FAQ (general)**: <https://www.openbsd.org/faq/>
- **Man pages**: `man pf.conf`, `man pfctl`, `man dhcpd.conf`, `man unbound.conf`, `man hostname.if`, `man afterboot` (a great post-install checklist)
- **Book: *The Book of PF*** by Peter N. M. Hansteen, the standard PF reference
- **OpenBSD errata (security patches)**: <https://www.openbsd.org/errata.html>
