# Threat Model

Security analysis of Qubes OS setup, threat actors, attack scenarios, and mitigations.

## Assets

**Tier 1 (Critical):**
- Credentials: KeePassXC database (vault)
- SSH private keys (sys-ssh-agent)
- GPG private keys (sys-pgp)
- Source code repositories (sys-git, dev)

**Tier 2 (High value):**
- Development environment configurations (dev)
- Documents (learning)
- Session tokens (browser cookies)

**Tier 3 (Medium value):**
- System configurations (Salt states)
- Network credentials (WireGuard config)

## Threat Actors

**Opportunistic attackers:**
- Skill level: Low to medium
- Likelihood: High
- Attack vectors: Malware, phishing, drive-by downloads

**Targeted attackers:**
- Skill level: Medium to high
- Likelihood: Medium
- Attack vectors: Spearphishing, supply chain, exploit chains

**APT:**
- Skill level: High
- Likelihood: Low (not a high-value target)
- Scope: Out of scope for this threat model

## Attack Scenarios

### Scenario 1: Browser Exploitation

**Attack vector:**
- Malicious website exploits browser vulnerability
- Compromise: browser qube

**Impact:**
- Assets at risk: Browser cookies, session tokens
- Blast radius: Limited to browser qube

**Mitigations:**
- Isolation: browser qube isolated from other qubes
- Network: VPN-only (sys-mullvad)
- Credentials: No credentials stored (vault offline)
- DispVM: Use disp-browser for untrusted sites

**Residual risk:**
- Medium: Session hijacking within browser qube
- Low: Credential theft (requires vault access)

### Scenario 2: Development Tool Compromise

**Attack vector:**
- Malicious npm package, PyPI package, or VSCodium extension
- Compromise: dev qube

**Impact:**
- Assets at risk: Source code, development credentials
- Blast radius: dev qube + sys-git (via Qrexec)

**Mitigations:**
- Isolation: dev qube isolated from vault, sys-pgp
- Git: Qrexec to sys-git (no direct network access)
- SSH: Qrexec to sys-ssh-agent (keys not accessible)
- Backup: Source code backed up

**Residual risk:**
- High: Source code exfiltration via network
- Medium: Git repository tampering
- Low: SSH key theft (sys-ssh-agent isolated)

### Scenario 3: Network Qube Compromise

**Attack vector:**
- Exploit in sys-net, sys-firewall, or sys-mullvad
- Compromise: Network infrastructure qube

**Impact:**
- Assets at risk: Network traffic metadata, VPN credentials
- Blast radius: All network-connected qubes (traffic visibility)

**Mitigations:**
- Templates: Minimal (reduced attack surface)
- Disposable variants: Available
- Airgapped qubes: Unaffected (sys-git, sys-pgp, sys-ssh-agent, vault)

**Residual risk:**
- High: Network traffic metadata visible
- Medium: VPN credentials exposed
- Low: AppVM data access (Xen isolation)

### Scenario 4: Qrexec Service Exploitation

**Attack vector:**
- Vulnerability in Git*, Ssh*, or qubes.Gpg2 RPC service
- Compromise: Service qube

**Impact:**
- Assets at risk: Git repos, SSH keys, GPG keys
- Blast radius: Critical (credential exposure)

**Mitigations:**
- Isolation: Service qubes airgapped
- Policy: Qrexec policies enforce ask prompts
- Backup: Keys/repos backed up externally

**Residual risk:**
- Critical: Key/credential theft if service compromised
- Medium: Policy bypass (requires Qrexec vulnerability)

### Scenario 5: Template Compromise

**Attack vector:**
- Malicious package in Qubes OS repositories
- Compromise: Template VM

**Impact:**
- Blast radius: High (multiple qubes)

**Mitigations:**
- Official repos: Qubes OS, Debian, Fedora only
- Verification: Package signatures verified
- Updates: Via dvm-mgmt (isolated)
- Minimal packages: Reduced attack surface

**Residual risk:**
- High: If official repo compromised, widespread impact

### Scenario 6: Dom0 Compromise

**Attack vector:**
- Vulnerability in Xen, Qubes OS, or dom0 services
- Compromise: dom0

**Impact:**
- Blast radius: Total

**Mitigations:**
- Xen isolation: Hypervisor-level VM separation
- dom0 hardening: Minimal software, no network
- Physical security: Disk encryption (LUKS)

**Residual risk:**
- Critical: Complete system compromise
- Out of scope: APT-level attacks

### Scenario 7: Physical Access

**Attack vector:**
- Stolen/lost laptop
- Evil Maid attack

**Impact:**
- Blast radius: Total (if disk decrypted)

**Mitigations:**
- Disk encryption: LUKS full-disk encryption
- Screen lock: Auto-lock after inactivity
- Backup: External backups

**Residual risk:**
- High: Cold boot attack (if powered on)
- Medium: Brute force disk encryption (weak passphrase)

## Mitigations

**Xen isolation:**
- VM separation via hypervisor
- Hardware-enforced memory isolation

**Network isolation:**
- VPN-only internet (sys-mullvad)
- Airgapped service qubes
- Firewall rules

**Qrexec policy enforcement:**
- Deny-by-default policies
- Ask prompts for sensitive operations

**Minimal attack surface:**
- Minimal templates
- Disabled services (cups, meminfo-writer)
- Minimal packages

**Split services:**
- Split-GPG2 (sys-pgp)
- Split-SSH (sys-ssh-agent)
- Split-Git (sys-git)

**Disposable VMs:**
- disp-browser (untrusted browsing)
- dvm-mgmt (template updates)

**Backup and recovery:**
- Vault: KeePassXC backed up
- Dev: Source code backed up
- Templates: Recreatable via Salt

## Risk Acceptance

**Accepted risks:**

1. **VPN provider trust:**
   - Risk: Mullvad sees all internet traffic
   - Justification: Paid with crypto, no logs policy

2. **Template supply chain:**
   - Risk: Malicious package in official repos
   - Justification: Low likelihood, official repos trusted

3. **Dom0 compromise:**
   - Risk: Xen or Qubes OS vulnerability
   - Justification: Out of scope (APT-level)

4. **Physical access:**
   - Risk: Stolen device, Evil Maid
   - Justification: Disk encryption sufficient for opportunistic attacks

5. **Network traffic correlation:**
   - Risk: Single VPN endpoint
   - Justification: Anonymity not primary goal

## Out of Scope

- APT attacks (nation-state, zero-days)
- Hardware implants (supply chain)
- Side-channel attacks (Spectre, Meltdown)
- Social engineering
- Xen hypervisor vulnerabilities (assumed secure)

## Security Review Cadence

**Quarterly review:**
- Threat model update
- Policy review
- Attack surface analysis
- Backup verification

**After major changes:**
- New qube deployment
- Template updates (major version)
- Network architecture changes

**Continuous:**
- Security updates
- Qrexec policy audits
- Backup testing

## Defense in Depth Layers

**Layer 1: Xen hypervisor**
- VM isolation
- Hardware-enforced memory separation

**Layer 2: Qube isolation**
- Network isolation (airgapped vs. VPN-only)
- Service isolation (split GPG, SSH, Git)

**Layer 3: Qrexec policy enforcement**
- Deny-by-default
- Ask prompts for sensitive operations

**Layer 4: Minimal attack surface**
- Minimal templates
- Disabled services
- Minimal packages

**Layer 5: Network security**
- VPN-only internet
- Firewall rules
- DNS enforcement

**Layer 6: Backup and recovery**
- Critical data backed up
- Templates recreatable via Salt

## Incident Response

**Browser compromise:**
1. Shutdown browser qube
2. Restore from template
3. Review vault for credential changes
4. Change passwords if necessary

**Development qube compromise:**
1. Shutdown dev qube
2. Review sys-git for unauthorized commits
3. Review network logs
4. Restore from backup
5. Audit source code

**Network qube compromise:**
1. Switch to disposable variant
2. Review network logs
3. Rotate VPN credentials
4. Recreate from template

**Service qube compromise:**
1. Shutdown immediately
2. Assume credentials compromised
3. Rotate all keys/credentials
4. Restore from backup

**Dom0 compromise:**
1. Assume total compromise
2. Full Qubes OS reinstall
3. Restore qubes from backup
4. Rotate all credentials