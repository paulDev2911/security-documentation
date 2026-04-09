Threat Model - Mobile Security

Last Updated: 2026-01-19 
Review: Undefined or on incident

---

Assets

Tier 1: Critical
- Banking credentials & access
- Investment accounts
- Crypto wallets (BTC, XMR)
- 2FA seeds (Aegis)
- Master passwords (KeePass)

Protection:
- Profile isolation (Banking profile)
- Local-only storage
- Encrypted backups
- Future: Dedicated device (Pixel 9 vault)

Tier 2: High-Value
- Signal communications
- Personal data
- Device integrity

Protection:
- GrapheneOS hardening
- Permission controls
- Storage scopes

Accepted Risk: Same device as daily use (profile isolation sufficient)

---

Threat Actors

Primary: Opportunistic Attacks
Likelihood: High | Sophistication: Low-Medium

Vectors:
- Malicious apps
- Phishing
- Public Wi-Fi exploitation
- Device theft

Mitigations:
- GrapheneOS exploit protections
- Obtainium (trusted sources)
- Mullvad VPN + kill switch
- PIN-only (no biometrics)
- Profile isolation

Risk: Low (after mitigations)

Secondary: Targeted Attacks
Likelihood: Low | Sophistication: Medium-High

Vectors:
- Spear phishing
- Physical access + coercion
- Supply chain (app updates)

Mitigations:
- Threat awareness
- PIN-only (no biometric coercion)
- Obtainium direct sources
- Future: Physical device separation

Risk: Low-Medium

Out of Scope: APT / Nation-State
Likelihood: Very Low | **Sophistication:** Very High

Rationale: Not a high-value target. High-sophistication defenses implemented for learning/portfolio, not necessity.

---

Attack Scenarios

Malicious App
Containment: Profile isolation, storage scopes, network permissions  
Response: Uninstall → Review data → Change credentials if banking affected → Restore from backup

Device Theft
Containment: 8-digit PIN per profile, encryption at rest  
Response: Assume compromised after 24h → Change all Tier 1 credentials → Revoke sessions → Restore to new device  
Future: Banking device in Faraday bag, on-demand only

Network Attack (MITM)
Containment: VPN + kill switch, certificate pinning  
Response: Disconnect → Verify VPN → Review traffic → Change credentials if needed

Phishing
Containment: KeePass (no credential reuse), 2FA, awareness  
Response: Verify legitimacy → Manual navigation (no links) → Change credentials if entered → Review accounts

---

Risk Acceptance

Banking + Main on Same Device
Risk: Single compromise = full loss  
Justification: Profile isolation adequate, temporary until OEM device  
Review Trigger: OEM release

Manual Backups (KeePass, Aegis)
Risk: Data loss on device + backup failure  
justification: Multiple backup locations (Proton Drive, homelab, paper codes)  
Review Trigger: Backup failure or loss incident

No Hardware Security Keys
Risk: Software 2FA compromise  
Justification: Sufficient for current threat model, added complexity vs. low likelihood  
Review Trigger: Threat sophistication increase
