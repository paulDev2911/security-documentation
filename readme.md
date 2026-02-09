# Security Documentation

Personal security infrastructure documentation covering mobile, desktop, and server compartmentalization.

## Overview

Documentation of my security-focused computing environment. Covers threat models, architecture decisions, hardening practices, and operational procedures across different platforms.

## Contents

### [Mobile Security](docs/mobile-security/)
GrapheneOS-based mobile compartmentalization using profile isolation.

- Threat model and risk assessment
- Profile architecture (Main, Banking, GPS)
- GrapheneOS hardening configuration
- Network security and VPN setup
- Physical separation strategy

### [Qubes OS](docs/qubes-os/)
Qubes OS desktop environment with Salt-based automation.

- VM structure and network isolation
- Salt states for infrastructure automation
- Split-GPG, Split-SSH, Split-Git hardening
- VPN integration (Mullvad WireGuard)
- Libreboot + Anti Evil Maid setup

### Server Infrastructure *(coming soon)*
Homelab and server security architecture.

- Planned: Network segmentation
- Planned: Service isolation
- Planned: Backup strategy
- Planned: Monitoring and logging

## Purpose

**Primary:** Personal reference and operational documentation

**Secondary:** Portfolio for IT security career progression

**Audience:** Future employers, security community, self

## Philosophy

Documentation is concise and technical. No fluff, straight to the point. Focus on concepts, strategies, and learnings rather than exhaustive step-by-step guides.

## Structure
```
docs/
├── mobile-security/      # GrapheneOS compartmentalization
│   ├── README.md
│   ├── architecture/
│   ├── devices/
│   └── hardening/
├── qubes-os/            # Qubes OS infrastructure
│   ├── README.md
│   ├── architecture/
│   ├── device/
│   ├── hardening/
│   └── salt/
└── server/              # Server infrastructure (planned)
    └── (coming soon)
```

## Status

- ✅ Mobile Security - Complete
- ✅ Qubes OS - Complete
- 🔲 Server Infrastructure - Planned

## License

Private repository. Not licensed for public use or distribution.