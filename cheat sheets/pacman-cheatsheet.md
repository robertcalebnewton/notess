# Pacman Cheat Sheet — Advanced Reference

A comprehensive, no-fluff reference for `pacman` on Arch Linux. Assumes you already know your way around a terminal.

---

## 0. Anatomy of a Command

`pacman` uses **operations** (what to do) plus **modifiers** (how to do it). Flags stack: `-Syu`, `-Rns`, `-Qdtq`, etc.

| Operation | Meaning                                                              |
| --------- | -------------------------------------------------------------------- |
| `-S`      | Sync — install from repos                                            |
| `-R`      | Remove                                                               |
| `-Q`      | Query — inspect installed packages                                   |
| `-U`      | Upgrade — install from a local file or URL                           |
| `-F`      | Files — query the file database (which package owns/contains a file) |
| `-D`      | Database — modify package metadata (install reason, etc.)            |
| `-T`      | Test dependencies (rarely used directly)                             |
| `-V`      | Version info                                                         |

---

## 1. Installing & Removing

```bash
pacman -S pkg                 # install package
pacman -S pkg1 pkg2 pkg3      # install multiple
pacman -S --needed pkg        # skip reinstall if already current — ALWAYS use this in scripts
pacman -U /path/to/file.pkg.tar.zst   # install a local package file
pacman -U https://example.com/pkg.pkg.tar.zst  # install directly from URL
```

Removal — know the difference, these are not interchangeable:

```bash
pacman -R pkg      # remove ONLY this package, leaves now-unneeded deps behind
pacman -Rs pkg      # remove package + dependencies no longer needed by anything else
pacman -Rns pkg     # -Rs + purge config files (no .pacsave). The "nuke it" combo.
pacman -Rc pkg      # remove package + everything that depends on it (cascade — dangerous)
pacman -Rdd pkg     # remove ignoring dependency checks entirely (breaks things if misused)
```

---

## 2. Querying Installed Packages

```bash
pacman -Q                 # list everything installed
pacman -Qi pkg             # detailed info (deps, size, install date, reason)
pacman -Qii pkg            # same + lists backup/config files
pacman -Ql pkg              # list every file owned by a package
pacman -Qo /path/to/file    # reverse lookup: which package owns this file?
pacman -Qs string           # search installed packages by name/description
pacman -Qk pkg / -Qkk pkg   # verify installed files against checksums (kk = thorough, checks perms too)
```

Install-reason filters (this is where advanced usage lives):

```bash
pacman -Qe     # explicitly installed (you asked for these)
pacman -Qd     # installed only as a dependency
pacman -Qdt    # orphans — deps that nothing depends on anymore
pacman -Qtdq   # same, quiet/name-only — feeds directly into removal (see §6)
pacman -Qm     # "foreign" packages — installed but NOT in any sync repo (i.e. AUR/manual)
pacman -Qn     # native packages — the inverse of -Qm
```

---

## 3. Searching & Inspecting Repos (not-yet-installed)

```bash
pacman -Ss string        # search repo package names/descriptions
pacman -Si pkg            # info about a package in the repos
pacman -Sl reponame       # list every package in a specific repo (core, extra, multilib...)
pacman -Sg                # list all package groups
pacman -Sg groupname       # list members of a group (e.g. base-devel, gnome)
pacman -Sii pkg            # like -Si but also shows reverse deps
```

File-based lookups (needs `pacman -Fy` once to build the files DB):

```bash
pacman -Fy                # sync the files database (separate from -Sy!)
pacman -F filename         # which package(s) provide this file/binary
pacman -Fl pkg              # list files inside a repo package, without installing it
pacman -Fo /path/to/file    # remote equivalent of -Qo, for packages you haven't installed
```

---

## 4. System Upgrades — Do This Right

```bash
pacman -Syu           # THE command: sync all repo databases + full upgrade. Use this, always.
pacman -Syyu          # force re-sync even if databases seem current (fixes "stale mirror" 404s)
```

**⚠️ Never do partial upgrades.** `pacman -Sy pkg` (sync without immediately following through with a full `-u`) desynchronizes your system against the repo state and *will* eventually break shared libraries. If you only take one rule from this sheet, it's this one. If you must delay a full upgrade, don't sync in the meantime.

```bash
pacman -Syu --ignore pkgname     # skip one package for this run only
pacman -Sw pkg                    # download only, don't install — pre-fills cache
pacman -Syuw                       # sync + download-only upgrade, install later
```

Permanent ignores go in `/etc/pacman.conf`:
```ini
IgnorePkg = pkgname
IgnoreGroup = groupname
```

---

## 5. Cache Management & Downgrading

Cache lives at `/var/cache/pacman/pkg/`.

```bash
pacman -Sc      # clean cache, keeps installed pkgs' current + one prior version (interactive)
pacman -Scc     # nuke ALL cached packages — you lose the ability to downgrade offline
```

Finer control via `paccache` (from `pacman-contrib`):

```bash
paccache -r          # keep the 3 most recent versions of each package (default)
paccache -rk1         # keep only 1 version per package
paccache -ruk0         # remove ALL cached versions of packages no longer installed
```

Downgrading:

```bash
pacman -U /var/cache/pacman/pkg/pkgname-OLDVERSION-x86_64.pkg.tar.zst
```
If it's not in your cache anymore, pull it from the **Arch Linux Archive**:
```bash
pacman -U https://archive.archlinux.org/packages/p/pkgname/pkgname-OLDVERSION-x86_64.pkg.tar.zst
```
(The AUR `downgrade` helper automates browsing the archive if you want it.)

---

## 6. Orphans & Dependency Management

```bash
pactree pkg          # dependency tree for a package (needs pacman-contrib)
pactree -r pkg         # REVERSE tree — what depends on this package
```

The classic orphan-cleanup one-liner:
```bash
pacman -Rns $(pacman -Qtdq)
```

Change a package's install reason without reinstalling it (useful after manually installing something that's actually a dependency, or vice versa):
```bash
pacman -D --asdeps pkg       # mark as a dependency
pacman -D --asexplicit pkg   # mark as explicitly installed (protects it from orphan cleanup)
```

---

## 7. Conflicts, Overwrites, and .pacnew Files

Force pacman to overwrite files it thinks conflict with another package (common with "file exists" errors on manual file placement):
```bash
pacman -S pkg --overwrite '*'
```

**`.pacnew` / `.pacsave` files** — pacman never silently overwrites a config file you've edited:
- Upgrade + you've modified `/etc/foo.conf` → new default is written as `/etc/foo.conf.pacnew`
- Removal + you've modified a config → old file is kept as `/etc/foo.conf.pacsave`

Find and merge them:
```bash
find /etc -name "*.pacnew"
sudo DIFFPROG=vimdiff pacdiff     # from pacman-contrib, walks you through merging each one
```

---

## 8. GPG Keyring Management

```bash
pacman-key --init                # initialize the keyring (fresh installs only)
pacman-key --populate archlinux    # load Arch's default trusted keys
pacman-key --refresh-keys          # pull latest keys from keyserver — fixes most "unknown trust" errors
pacman-key --lsign-key KEYID       # locally sign/trust a specific key (needed for some third-party repos)
```

Nuclear option for a corrupted keyring:
```bash
sudo rm -rf /etc/pacman.d/gnupg
sudo pacman-key --init
sudo pacman-key --populate archlinux
```

---

## 9. Mirror Optimization

Mirrors are listed in `/etc/pacman.d/mirrorlist`. Use `reflector` to keep it fast:
```bash
reflector --country 'US' --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
reflector --latest 20 --sort rate --save /etc/pacman.d/mirrorlist
```
Run this whenever you get inexplicable 404s or downloads feel slow — it's almost always a stale/overloaded mirror.

---

## 10. Key `pacman.conf` Settings

```ini
[options]
Architecture = auto
ParallelDownloads = 5      # download multiple packages concurrently — big speed win
Color
ILoveCandy                  # progress bar easter egg, cosmetic only
CheckSpace
SigLevel    = Required DatabaseOptional
IgnorePkg   = pkgname
IgnoreGroup = groupname

[multilib]                  # uncomment to enable 32-bit packages (Steam, Wine, etc.)
Include = /etc/pacman.d/mirrorlist
```
Repo order in the file matters: for identically-named packages across repos, the **first** repo listed wins. Custom/local repos usually go above `[core]`.

---

## 11. Pacman Hooks

Located in `/etc/pacman.d/hooks/*.hook` (user-defined) and `/usr/share/libalpm/hooks/` (package-provided). Hooks run scripts `Pre`/`PostTransaction` on matching package operations — this is how things like automatic `mkinitcpio` rebuilds or `grub-mkconfig` updates happen after a kernel upgrade. Write your own `.hook` file if you need a script to fire automatically on install/upgrade/removal of specific packages.

---

## 12. Logs & Recovery

```bash
/var/log/pacman.log                          # full transaction history
grep -i "installed\|upgraded\|removed" /var/log/pacman.log
```

Common failure states:
```bash
# "failed to init transaction: unable to lock database"
# — only if pacman ISN'T actually running elsewhere:
sudo rm /var/lib/pacman/db.lck

# recovering a broken system from the Arch ISO:
mount /dev/sdXn /mnt
arch-chroot /mnt
pacman -Syu
```

---

## 13. Testing Repos

`[core-testing]`, `[extra-testing]`, `[multilib-testing]` — enable these in `pacman.conf` (above their stable counterparts) if you want bleeding-edge packages before they hit `[core]`/`[extra]`. Expect occasional breakage; this is how you'd help catch bugs pre-release.

---

## 14. Power One-Liners

```bash
# Remove all orphans
pacman -Rns $(pacman -Qtdq)

# 20 largest installed packages by size
pacman -Qi | awk '/^Name/{n=$3} /^Installed Size/{print $4$5, n}' | sort -rh | head -20

# List explicitly-installed packages only (your "real" install list, good for reproducing a setup)
pacman -Qeq > my-packages.txt

# List AUR/foreign packages (for AUR helper bulk updates or auditing)
pacman -Qmq

# Check for updates WITHOUT touching the live database (safe for scripts, status bars, etc.)
checkupdates
```

---

## 15. Companion Tools Worth Installing

None of these are `pacman` itself, but any advanced Arch setup leans on them:

| Tool | Package | What it's for |
|---|---|---|
| `paccache`, `pactree`, `pacdiff`, `checkupdates` | `pacman-contrib` | cache trimming, dep trees, pacnew merging, safe update checks |
| `reflector` | `reflector` | mirrorlist optimization |
| `expac` | `expac` | printf-style custom formatting for pacman queries |
| `pkgfile` | `pkgfile` | find which *uninstalled* package provides a binary (`command-not-found` style) |
| `arch-audit` | `arch-audit` | cross-checks installed packages against known CVEs |

---

## 16. Golden Rules

1. **Never partial-upgrade.** `-Sy` without an immediate `-u` is the #1 cause of "my Arch install randomly broke."
2. Always use `--needed` when scripting installs — avoids pointless reinstalls.
3. Don't `-Scc` unless you're truly out of disk space — you lose downgrade capability.
4. Check `/etc/*.pacnew` after every significant upgrade, especially for services you run (nginx, sshd, systemd units).
5. Keep `reflector` on a periodic timer if you're on a laptop that changes networks/regions often.
6. `-Rdd` and `-Sdd`(force, ignoring deps) exist for emergencies — they are not for routine use.
