# Mobile Compartmentalization

> Physical device separation strategy for mobile security: Implementing compartmentalization principles across multiple GrapheneOS devices

## Overview

A practical implementation of physical device compartmentalization for mobile security, based on threat modeling, least-privilege principles, and defense-in-depth architecture.

**Core Philosophy:** Separate physical devices for different risk profiles prevent total compromise from a single attack vector. Not motivated by paranoia, but by logic: *my device, my data, my decisions.*

## Current Architecture

### Device 1: Pixel 9 (Primary - Temporary)
**Purpose:** All-in-one device until GrapheneOS OEM arrives  
**Status:** Production  
**OS:** GrapheneOS (since official Pixel 9 support)

**Profiles:**
- **Owner:** Empty (system management only)
- **Main:** Daily apps, communications, general use
- **Banking:** Financial apps, investments, crypto
- **GPS:** Apps requiring Google Play Services (isolated)

**Philosophy:** Profile-based compartmentalization as interim solution until physical device separation.

### Device 2: GrapheneOS OEM (Planned - Future Primary)
**Purpose:** Daily driver & communications  
**Status:** Awaiting release (Q4 2026 / Q1 2027?)
**Rationale:** 
- Non-Google hardware for ideological consistency
- Snapdragon performance > Tensor
- Official GrapheneOS support maintained

**Planned Use:**
- Main profile migration
- GPS profile migration
- Daily communications
- General use apps

### Device 3: Pixel 9 (Future Secondary - Banking Vault)
**Purpose:** Financial operations only  
**Status:** Will transition to dedicated banking device  

**Planned Configuration:**
- Owner profile only (minimal)
- Banking profile only
- Maximum hardening (Airplane mode default, Faraday bag storage)
- Minimal usage = minimal exposure
- Physical separation from daily device

## 🔐 Security Architecture

### Current State (Single Device)
```
          Pixel 9 (GrapheneOS)

Owner Profile      (Empty)

Main Profile       Daily Use
                     • Signal/Comms     
                     • NewPipe          
                     • Infinity (X)     
                     • KeepassDX (local)
                     • Aegis (2FA)      

Banking Profile    Financial          
                    • Bank apps        
                    • Investment apps  
                    • Crypto (BTC/XMR) 
GPS Profile        Isolated           
                     • Sandboxed Play   
                     • Apps requiring 
                       Google Services  


Acceptable Risk: Banking + Main on same device
Mitigation: Profile isolation, hardening
```

### Future State (Physical Separation)
```

GrapheneOS OEM -> Daily Driver
- Main Profile
- GPS Profile
- Daily Usw
- Communications
High Exposure, medium risk and a 24/7 usage


Pixel 9 -> Banking Vault
- Banking Only
- 2FA (Aegis)
- Passwords
- Investments
minimal exposure, high value and On-Demand only

=> Physical Device Seperation (Containment)

Risk Reduction: Device compromise ≠ total loss
```

## Security Features

### GrapheneOS Hardening
- Verified boot with custom keys
- Exploit mitigation (MTE, PAC, BTI)
- Sandboxed Google Play (isolated to GPS profile)
- User profile isolation
- Network permission controls (per-app, on-demand)
- Storage Scopes (dedicated folder)
- Sensor permissions (disabled by default)

### Access Control
- Separate 8-digit random PIN per profile
- No biometrics (cannot be coerced)
- Least Privilege principle (network/storage/sensors off unless needed)
- Contact access disabled (except Signal - full access)

### Network Security
- Mullvad VPN (paid with crypto)
- Always-on VPN with kill switch
- Split-tunneling available (manual, rare use)

### Data Protection
- KeePass: Local database, manual sync via Proton Drive
- Keyfile: Split across 2 USB sticks (1 on person, 1 secure storage)
- Aegis: Local database (no cloud sync)
- 2FA backup codes: Paper + encrypted digital backup
- GrapheneOS native backup → Homelab + NAS (monthly, or on major changes)

## App Management

### Installation Sources
**Primary:** Obtainium (centralized updates)
- Sources: GitHub, F-Droid repos, direct APKs from websites
- Single-app update management
- No Google Play dependency

**Fallback:** Aurora Store (avoided when possible)

**Sandboxed Google Play:** GPS profile only (for apps requiring GMS)

### App Distribution

**Main Profile:**
- NewPipe (YouTube frontend)
- Infinity (X/Twitter client)
- Signal (full contact access)
- [Other daily apps]
- Aegis (2FA)
- KeePassDX (local password database)

**Banking Profile:**
- Bank apps (requires sandboxed Play)
- Investment platform apps
- Crypto apps (Bitcoin, Monero, Bitpanda, etc.)

**GPS Profile:**
- Apps requiring Google Play Services (isolated)

## Threat Model

### Assets to Protect
**Tier 1 (Critical):**
- Banking access & credentials
- Investment accounts
- Crypto wallets & keys
- 2FA seeds
- Master passwords

**Tier 2 (High Value):**
- Private communications (acceptable risk)
- Personal data
- Device integrity

### Threat Actors
**Primary:** Opportunistic malware, phishing, credential theft  
**Mitigation:** Hardened OS, profile isolation, least privilege

**Not Targeted:** Advanced persistent threats, nation-state actors  
**Note:** High-sophistication defenses implemented out of interest and as learning exercise, not necessity

### Acceptable Risks
- Banking + Main on same device (current state)
  - *Mitigated by:* Profile isolation, minimal app surface, hardening
  - *Future:* Physical separation when OEM device arrives
- Manual KeePass & Aegis backups
  - *Mitigated by:* Encrypted backups, paper backup codes

## Future: GrapheneOS OEM Device

**Status:** Tracking (Expected Q4 2026 / Q1 2027)

GrapheneOS is partnering with a major Android OEM to bring official support to flagship Snapdragon-powered devices.

**Why This Matters:**
- Non-Google hardware (ideological consistency)
- Better performance (Snapdragon > Tensor)
- Enables true physical device separation
- Maintains GrapheneOS security guarantees

**Migration Strategy:**
1. **Current:** All functions on Pixel 9
2. **OEM Release:** Migrate Main + GPS to OEM device
3. **Final State:** Pixel 9 becomes dedicated banking vault

See [OEM Device Tracking](docs/future/grapheneos-oem-tracking.md) for updates.

## Documentation

### Current Setup
- [Pixel 9 Configuration](docs/grapheneos-pixel9/01-current-setup.md)
- [Profile Architecture](docs/grapheneos-pixel9/02-profile-architecture.md)
- [Hardening Guide](docs/grapheneos-pixel9/03-hardening.md)
- [App Management](docs/grapheneos-pixel9/04-app-management.md)

### Security
- [Threat Model](docs/security/01-threat-model.md)
- [Attack Scenarios](docs/security/02-attack-scenarios.md)
- [Backup Strategy](docs/security/03-backup-strategy.md)

### Future Planning
- [GrapheneOS OEM Tracking](docs/future/grapheneos-oem-tracking.md)
- [Migration Strategy](docs/future/migration-strategy.md)
- [Physical Separation Plan](docs/future/physical-separation.md)

### Workflows
- [Banking Operations](docs/workflows/banking.md)
- [Daily Usage](docs/workflows/daily-usage.md)
- [Emergency Procedures](docs/workflows/emergency.md)

## Philosophy

**Core Principles:**
1. **Least Privilege:** Apps get minimum permissions needed, on-demand
2. **Defense in Depth:** Multiple security layers (OS, profiles, network, physical)
3. **Compartmentalization:** Separate contexts = contained blast radius
4. **Pragmatic Security:** Balance security with usability
5. **Self-Sovereignty:** My device, my data, my decisions

**Not Driven By:**
- Paranoia
- Tin-foil-hat thinking
- Unrealistic threat models

**Driven By:**
- Logical security principles
- Learning and skill development
- Personal interest in security engineering
- Portfolio building for IT security career

## Roadmap

### Current Phase (Dec 2024 - Q4 2026)
- [x] GrapheneOS Pixel 9 setup
- [x] Profile-based compartmentalization
- [x] Hardening implementation
- [ ] Complete documentation
- [ ] Homelab integration (Authentik, Pomerium, Headscale - planned)
- [ ] Zero-trust mobile access architecture

### Future Phase (Q4 2026 - Q1 2027)
- [ ] GrapheneOS OEM device acquisition
- [ ] Migration of Main + GPS profiles
- [ ] Pixel 9 reconfiguration as banking vault
- [ ] Maximum hardening on banking device
- [ ] Physical separation finalized

### Long-term
- [ ] Homelab mobile access (Nextcloud, Paperless, Authentik)
- [ ] Zero-trust architecture implementation
- [ ] Community contributions
- [ ] Blog posts / knowledge sharing

## ⚠Disclaimer

This documentation represents a personal security implementation based on specific threat modeling and use cases. Your requirements will differ.

**Not security advice.** Conduct your own threat modeling and risk assessment.

## License

MIT License - Documentation and configurations are provided as-is for educational purposes.

## Purpose

**Primary:** Personal reference and progress tracking  
**Secondary:** Portfolio demonstration for IT security career  
**Tertiary:** Community knowledge sharing (when made public)

**Documentation Goal:** Professional, comprehensive, but not exhaustively detailed on every minor point. Focus on concepts, strategies, and learnings.

---

**Last Updated:** 2025-01-12  
**Status:** Active Development  
**Next Milestone:** Complete Pixel 9 documentation