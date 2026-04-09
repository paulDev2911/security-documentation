Profile Isolation Architecture

Last Updated: 2026-01-19

---

Profile Structure

Owner Profile
Purpose: System management only  
State: Empty (no apps)  
Rationale: Separation of admin/user contexts

Main Profile
Purpose: Daily use & communications  
Apps:
- Signal (full contact access)
- KeePassDX (local DB)
- Aegis (2FA)
- different Frontend apps

Permissions: Minimal, on-demand only, deny by default
Network: Mullvad VPN always-on

Banking Profile
Purpose: Financial operations only  
Apps:
- Bank apps
- Investment platforms
- Crypto (BTC, XMR)
- Keepass and Aegis
- Google Play Services (Sandboxed)

Permissions: Minimal, isolated from Main, deny by default
Network: VPN enforced, no split-tunneling

GPS Profile
Purpose: Apps requiring Google Play Services  
Implementation: Sandboxed Play (isolated)  
Rationale: Containment of Google dependencies

---

Security Controls

Per-Profile:
- Separate 8-digit random PIN
- Independent network permissions
- Isolated storage (Storage Scopes)
- Separate sensor access

Cross-Profile Protection:
- No shared clipboard
- No file sharing between profiles
- Process isolation enforced by GrapheneOS

---

Blast Radius Containment

Compromise Scenario:
- Main Profile compromised → Banking + GPS unaffected
- GPS Profile compromised → Main + Banking unaffected
- Banking Profile compromised → Main + GPS unaffected

Limitation: Single physical device = kernel-level exploit affects all profiles

Future Mitigation: Physical device separation (see `physical-separation.md`)

---

Status: Production  
Next Step: Physical separation on OEM device release