Physical Separation Plan

Last Updated: 2026-01-19
Status: Planned (awaiting OEM device)

---

Target Architecture

Device 1: GrapheneOS OEM (Daily Driver)
Purpose: Daily use + communications  
Profiles:
- Main (Signal, NewPipe, daily apps)
- GPS (sandboxed Play Services)
- others

Exposure: High (24/7 usage)  
Risk Profile: Medium  
Mitigation: Profile isolation, hardening

Device 2: Pixel 9 (Banking Vault)
Purpose: Financial operations only  
Profiles:
- Owner (minimal)
- Banking (financial apps only)

Exposure: Minimal (on-demand only)  
Risk Profile: High-value, low-exposure  
Mitigations:
- Airplane mode default
- Faraday bag storage
- Physical separation from daily device

---

Security Advantage

Current (Single Device):
- Profile isolation = software boundary
- Kernel exploit = all profiles compromised

Future (Physical Separation):
- Device compromise ≠ total loss
- Banking device minimal attack surface (rare usage)
- Independent exploit chains required

---

Migration Strategy

Phase 1: OEM device arrives (Q4 2026 / Q1 2027)  
Phase 2: Migrate Main + GPS profiles to OEM  
Phase 3: Wipe Pixel 9, reconfigure as Banking vault  
Phase 4: Harden Pixel 9 (airplane mode, Faraday bag, on-demand only)

---

Trade-offs

Benefits:
- Physical containment
- Reduced banking device exposure
- Independent failure domains

Costs:
- Carry two devices (situational - banking device on-demand only)
- Sync complexity (minimal - no cross-device data sharing)

Verdict: Security gain outweighs minor inconvenience for Tier 1 assets