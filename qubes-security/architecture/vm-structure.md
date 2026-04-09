#VM Structure

Qube organization based on function and security level.

##Infrastructure Qubes

**sys-net (HVM):**
- Template: debian-minimal
- Purpose: Hardware network access (PCI device passthrough)
- Network: Direct internet access
- Features: servicevm, provides-network
- Disposable variant: disp-sys-net
- Autostart: Yes
- Storage: Default (system only)
- Backup: No

**sys-firewall (ProxyVM):**
- Template: debian-minimal
- Purpose: Network filtering, DNS enforcement
- Network: sys-net → sys-firewall
- Features: servicevm, provides-network
- Disposable variant: disp-sys-firewall
- Autostart: Yes
- Storage: Default
- Backup: No

**sys-mullvad (ProxyVM):**
- Template: fedora-minimal
- Purpose: WireGuard VPN endpoint
- Network: sys-firewall → sys-mullvad
- Features: servicevm, provides-network
- Firewall: Restrictive (VPN endpoint only)
- DNS: Mullvad DNS (10.64.0.1)
- Autostart: Yes
- Storage: Default
- Backup: No

**sys-usb (HVM):**
- Template: debian-minimal
- Purpose: USB device isolation
- Network: None (airgapped)
- Features: servicevm
- Disposable variant: disp-sys-usb
- Autostart: Yes
- Storage: Default
- Backup: No

**mgmt (DispVM template):**
- Template: fedora-minimal
- Purpose: Custom management DispVM (replaces default-mgmt-dvm)
- Network: sys-mullvad
- Usage: Template updates, Salt management
- Features: Custom management-dvm
- Autostart: No
- Storage: Default
- Backup: No

##Security Qubes

**sys-git (AppVM):**
- Template: tpl-sys-git (debian-minimal based)
- Purpose: Git server via Qrexec (GitFetch/GitPush/GitInit)
- Network: None (airgapped)
- Features: servicevm
- Services: Git* (RPC)
- Policy: ask @anyvm → sys-git (default)
- Autostart: Yes
- Storage: 20GB private volume
- Backup: No (repositories backed up externally)

**sys-pgp (AppVM):**
- Template: tpl-sys-pgp (debian-minimal based)
- Purpose: Split-GPG2 server (offline key storage)
- Network: None (airgapped)
- Features: servicevm
- Services: qubes.Gpg2
- Policy: Client-specific rules
- Autostart: No (on-demand)
- Storage: Default
- Backup: No (keys backed up externally)

**sys-ssh-agent (AppVM):**
- Template: tpl-sys-ssh-agent (debian-minimal based)
- Purpose: SSH agent server via Qrexec (multi-agent support)
- Network: None (airgapped)
- Features: servicevm
- Services: SshAgent+AGENT
- Policy: Per-agent rules (e.g., SshAgent+work @anyvm → sys-ssh-agent)
- Autostart: No (on-demand)
- Storage: 40GB private volume (SSH keys)
- Backup: No (keys backed up externally)

**sys-ssh (AppVM):**
- Template: tpl-sys-ssh (debian-minimal based)
- Purpose: SSH/SSHFS client via Qrexec
- Network: sys-mullvad (VPN required)
- Features: servicevm
- Services: Ssh
- Policy: ask @anyvm → sys-ssh
- Autostart: No (on-demand)
- Storage: 40GB private volume (~/.ssh/config, known_hosts)
- Backup: No

**vault (AppVM):**
- Template: tpl-vault (fedora-minimal based)
- Purpose: Offline credential storage (KeePassXC)
- Network: None (airgapped)
- Autostart: No (manual access only)
- Storage: Default
- Backup: Yes (critical - KeePassXC database)

##Work Qubes

**dev (AppVM):**
- Template: tpl-dev (fedora-xfce based)
- Purpose: Development environment
- Network: sys-mullvad
- Tools: VSCodium, .NET SDK, Node.js, Python, JetBrains Rider
- Git: Qrexec to sys-git
- SSH: Qrexec to sys-ssh (via sys-ssh-agent)
- Autostart: No
- Storage: Default
- Backup: Yes (source code, configs)

**learning (AppVM):**
- Template: tpl-learning (fedora-minimal based)
- Purpose: Document editing (OnlyOffice)
- Network: sys-mullvad
- Autostart: No
- Storage: Default
- Backup: Yes (documents)

**browser (AppVM):**
- Template: tpl-browser (fedora-minimal based)
- Purpose: General web browsing
- Network: sys-mullvad
- Browsers: Brave, Mullvad Browser, Chromium
- DispVM: disp-browser (for untrusted sites)
- Autostart: No
- Storage: Default
- Backup: No

##Templates

**Debian-based:**
- debian (Official, general purpose)
- debian-minimal (Infrastructure: sys-net, sys-firewall, sys-usb)
- debian-xfce (Optional, currently unused)
- tpl-sys-git (debian-minimal + git, qrexec services)
- tpl-sys-pgp (debian-minimal + gnupg2, split-gpg2)
- tpl-sys-ssh-agent (debian-minimal + openssh-server, qrexec services)
- tpl-sys-ssh (debian-minimal + openssh-client, qrexec services)

**Fedora-based:**
- fedora (Official, general purpose)
- fedora-minimal (Most custom templates)
- fedora-xfce (Development, management)
- tpl-browser (fedora-minimal + Brave, Mullvad, Chromium)
- tpl-dev (fedora-xfce + development tools)
- tpl-learning (fedora-minimal + OnlyOffice)
- tpl-vault (fedora-minimal + KeePassXC)
- tpl-sys-mullvad (fedora-minimal + wireguard-tools)

**Template management:**
- Updates via dvm-mgmt (custom management DispVM)
- Base templates: Backup disabled
- Custom templates: Cloned from minimal variants
- Hardening: Minimal packages, disabled services (cups, meminfo-writer)

##DispVM Templates

**dvm-mgmt:**
- Base: fedora-xfce
- Purpose: Management DispVM (replaces default)
- Network: sys-mullvad
- Used by: All qubes (template updates)

**dvm-browser:**
- Base: tpl-browser
- Purpose: Disposable browsing (untrusted sites)
- Network: sys-mullvad
- AppVM: disp-browser

**disp-sys-net:**
- Base: debian-minimal
- Purpose: Disposable network qube
- Network: Direct internet

**disp-sys-firewall:**
- Base: debian-minimal
- Purpose: Disposable firewall
- Network: sys-net

**disp-sys-usb:**
- Base: debian-minimal
- Purpose: Disposable USB handling
- Network: None

##Qube Relationships

**Network flow:**
```
Internet
  ↓
sys-net (HVM, PCI devices)
  ↓
sys-firewall (DNS enforcement)
  ↓
sys-mullvad (WireGuard VPN)
  ↓
├── dev
├── learning
├── browser
└── sys-ssh
```

**Service relationships:**
```
dev (client)
  ├── Git → sys-git (Qrexec: GitFetch/GitPush)
  ├── SSH → sys-ssh-agent (Qrexec: SshAgent+work)
  └── SSH → sys-ssh (Qrexec: Ssh)

sys-ssh (client)
  └── SSH Agent → sys-ssh-agent (Qrexec: SshAgent+work)

sys-git (server)
  └── Provides: Git repositories (offline)

sys-ssh-agent (server)
  └── Provides: SSH agent sockets (offline)

sys-ssh (client)
  └── Connects: External SSH servers (via VPN)
```

##Storage Allocation

**Default (system + private):**
- Infrastructure qubes: ~2-3GB system
- Templates: ~10GB system
- AppVMs: ~2GB system + variable private

**Custom private volumes:**
- sys-git: 20GB (Git repositories)
- sys-ssh-agent: 40GB (SSH keys, identities)
- sys-ssh: 40GB (SSH configs, known_hosts)

**Backup-enabled:**
- vault (critical: KeePassXC database)
- dev (source code, configs)
- learning (documents)

**Backup-disabled:**
- All templates (recreatable via Salt)
- All infrastructure qubes (recreatable via Salt)
- browser (disposable content)

##Resource Allocation

**Memory (MB):**
- sys-net: 300 (200 min, 300 max)
- sys-firewall: 300 (200 min, 300 max)
- sys-mullvad: 300 (200 min, 300 max)
- sys-usb: 200 (200 min, 300 max)
- sys-git: 300 (200 min, 300 max)
- sys-pgp: 300 (200 min, 300 max)
- sys-ssh-agent: 300 (200 min, 300 max)
- sys-ssh: 400 (300 min, 400 max)
- vault: 400 (300 min, 400 max)
- dev: 4000 (400 min, 4000 max)
- learning: 4000 (400 min, 4000 max)
- browser: 4000 (400 min, 4000 max)

**vCPUs:**
- Most qubes: 2 vCPUs
- Development/browser: 4 vCPUs

##Qube Features

**servicevm:**
- Purpose: Mark infrastructure/service qubes
- Enabled: sys-net, sys-firewall, sys-mullvad, sys-usb, sys-git, sys-pgp, sys-ssh-agent, sys-ssh
- Effect: Excluded from regular qube operations

**service.* features:**
- service.qubes-updates-proxy: Enabled on sys-firewall, sys-mullvad
- service.cups: Disabled on all qubes (printing not needed)
- service.cups-browsed: Disabled on all qubes
- service.meminfo-writer: Disabled on templates (reduces attack surface)

**Audio:**
- audiovm: "" (disabled) on all qubes except those requiring sound
- Rationale: Reduce attack surface, most qubes don't need audio

##Tags

**Created by Salt:**
- Infrastructure qubes: No special tags
- Service qubes: No special tags
- Work qubes: No special tags

**Potential future tags:**
- `backup-daily` for critical qubes
- `network-restricted` for limited network access
- `offline` for airgapped qubes

##Autostart Behavior

**Always autostart:**
- sys-net (required for network)
- sys-firewall (required for network)
- sys-mullvad (required for VPN)
- sys-usb (required for USB devices)
- sys-git (always-available Git server)

**On-demand (manual start):**
- vault (security: manual access only)
- sys-pgp (security: manual access only)
- sys-ssh-agent (security: manual access only)
- sys-ssh (on-demand SSH access)
- dev (on-demand development)
- learning (on-demand documents)
- browser (on-demand browsing)
- mgmt (DispVM template, not started directly)

##Security Boundaries

**Network isolation tiers:**
1. Airgapped (no network): sys-git, sys-pgp, sys-ssh-agent, vault, sys-usb
2. VPN-only: dev, learning, browser, sys-ssh
3. Firewall-only: (none currently)
4. Direct internet: sys-net

**Service isolation:**
- Git server: sys-git (offline, Qrexec-only access)
- GPG keys: sys-pgp (offline, Qrexec-only access)
- SSH keys: sys-ssh-agent (offline, Qrexec-only access)
- SSH client: sys-ssh (online via VPN, Qrexec-only access)

**Template isolation:**
- Base templates: Official Qubes OS repos only
- Custom templates: Cloned from base, minimal modifications
- DispVM templates: Separate from persistent AppVMs