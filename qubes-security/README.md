# Qubes OS Setup

Automated Qubes OS infrastructure using Salt for security-focused compartmentalization.

## Architecture

**Core Infrastructure:**
- sys-net, sys-firewall (Debian-minimal based, disposable variants available)
- sys-mullvad (WireGuard VPN, Fedora-minimal based)
- sys-usb (USB isolation, disposable variant)
- mgmt (Custom management DispVM, replaces default-mgmt-dvm)

**Security Qubes:**
- sys-git (Git server with Qrexec protocol: GitFetch/GitPush/GitInit)
- sys-pgp (Split-GPG2 server, offline)
- sys-ssh-agent (SSH agent server with Qrexec, multi-agent support)
- sys-ssh (SSH/SSHFS server with Qrexec)
- vault (KeePassXC, offline storage)

**Work Qubes:**
- dev (Development environment: VSCodium, .NET, Node.js, Python, Rider)
- learning (OnlyOffice for documents)
- browser (Brave, Mullvad Browser, Chromium with DispVM support)

## Templates

**Debian:**
- debian (Standard)
- debian-minimal (Infrastructure qubes)
- debian-xfce (Optional GUI)

**Fedora:**
- fedora (Standard)
- fedora-minimal (Most templates use this)
- fedora-xfce (Development, management)

## Network Isolation

```
Internet
  ↓
disp-sys-net (HVM, PCI devices)
  ↓
disp-sys-firewall (ProxyVM)
  ↓
sys-mullvad (VPN, WireGuard)
  ↓
AppVMs
```

## Qrexec Services

**Git (sys-git):**
- GitFetch (git-upload-pack)
- GitPush (git-receive-pack)
- GitInit (initialize bare repo)
- Client: `git clone qrexec://@default/repo-name`

**SSH (sys-ssh):**
- Ssh (SSH connection over Qrexec)
- Client: `ssh -p 1840 localhost` (socket forwarding)
- SSHFS: `sshfs -p 1840 localhost:/path /mount`

**SSH Agent (sys-ssh-agent):**
- SshAgent+AGENT (per-agent access)
- Client: `systemctl start ssh-agent-forwarder@work.service`
- Socket: `/tmp/ssh-agent-forwarder/work.sock`

**GPG (sys-pgp):**
- qubes.Gpg2 (Split-GPG2)
- Client: split-gpg2 integration
- Config: `~/.config/qubes-split-gpg2/qubes-split-gpg2.conf`

**Proxy (sys-net, sys-firewall, sys-mullvad):**
- ConnectTCP+host+port (TCP proxy via socat)
- Example: SSH ProxyCommand integration

## Salt Automation

**Deployment:**
```sh
cd ~/salt-qubes-sec
sudo ./scripts/setup.sh  #Deploy to /srv/user_salt/salt-qubes-sec
```

**Development workflow:**
```sh
#In dev qube: edit, commit, push to sys-git
#In dom0:
./scripts/sync-from-dev.sh dev  #Sync from dev VM
sudo ./scripts/setup.sh         #Deploy to Salt
sudo qubesctl state.show_top    #Verify
```

**Apply states:**
```sh
# Specific qube
sudo qubesctl --skip-dom0 --targets=tpl-dev state.apply dev.install

# All via top file
sudo qubesctl top.enable sys-git
sudo qubesctl --targets=tpl-sys-git,sys-git state.apply
sudo qubesctl top.disable sys-git
```

## Hardening

**Template hardening:**
- Minimal package installation (install_recommends: False)
- Audio disabled (audiovm: "")
- Unnecessary services disabled (cups, cups-browsed, meminfo-writer)
- Locale configuration (Debian templates)

**Network hardening:**
- sys-mullvad: Firewall rules restrict to VPN endpoint only
- sys-firewall: DNS redirection to VPN DNS
- Minimal network exposure (most qubes: netvm: "")

**Storage:**
- sys-git: 20GB private volume
- sys-ssh: 40GB private volume
- vault: Backup enabled
- Templates: Backup disabled

**Features:**
- servicevm flag on infrastructure qubes
- Split services (GPG, SSH, Git) isolated
- DispVMs for untrusted browsing

## File Structure

See [salt/README.md](salt/README.md) for detailed Salt structure documentation.

## Management

**Qube lifecycle:**
```sh
#Create via Salt
sudo qubesctl state.apply sys-git.create

#Template updates handled automatically via dvm-mgmt
#Custom management DispVM configured as global default

#Cleanup
sudo qubesctl state.apply sys-git.cleanup  #If exists
```

**Policy management:**
- Policies in `/etc/qubes/policy.d/80-<project>.policy`
- Source: `salt/<project>/files/admin/policy/default.policy`
- Macro: `policy_set(sls_path, '80')` in create.sls

**Template synchronization:**
- Appmenus: `sync_appmenus` macro
- Updates: `update_admin` macro (forces shutdown after update)

## Threat Model

**In-scope:**
- AppVM compromise (isolation via Xen, Qrexec)
- Network eavesdropping (VPN mandatory, split network qubes)
- Credential theft (offline vault, split-GPG/SSH)
- Malicious repositories (sys-git airgapped, Qrexec-only access)

**Out-of-scope:**
- Dom0 compromise
- Xen hypervisor vulnerabilities
- Hardware attacks (DMA, physical access)
- Supply chain attacks on Qubes OS itself

**Mitigations:**
- Network isolation (sys-net → sys-firewall → sys-mullvad)
- Service isolation (sys-git, sys-pgp, sys-ssh-agent offline)
- Disposable qubes for untrusted content
- Minimal attack surface (minimal templates, disabled services)
- Qrexec policy enforcement (deny by default)

## Backup Strategy

**Included in backups:**
- vault (KeePassXC database)
- dev (source code, configs)
- learning (documents)

**Excluded from backups:**
- All templates
- Infrastructure qubes (sys-*, mgmt)
- DispVM templates (dvm-*)
- browser (disposable content)

**Backup location:**
- Manual: Qubes Backup to external drive
- Frequency: After significant changes
- Encryption: Qubes Backup built-in encryption

## Recovery

**Template reinstall:**
```sh
sudo qubesctl state.apply <template>.clone
```

**Qube recreation:**
```sh
sudo qubesctl state.apply <qube>.create
sudo qubesctl --skip-dom0 --targets=tpl-<qube> state.apply <qube>.install
sudo qubesctl --skip-dom0 --targets=<qube> state.apply <qube>.configure
```

**Salt repository:**
- Primary: sys-git (`~/src/salt-qubes-sec.git`)
- Secondary: dev qube working copy
- Tertiary: External backup

## Future Improvements

- Whonix integration (sys-whonix for Tor)
- USB device policies (currently deny-all)
- Audio qube separation (currently using dom0)
- Automated backup scheduling
- sys-ssh-agent key expiration automation
- Qubes OS R4.3 migration testing