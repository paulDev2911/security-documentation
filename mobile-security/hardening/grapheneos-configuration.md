GrapheneOS Configuration

Last Updated: 2026-01-19
Device: Pixel 9
OS Version: GrapheneOS (always latest stable)

---

Boot & Verification

Bootloader: Locked (post-installation)
Verified Boot: Custom AVB keys
OEM Unlocking: Disabled

Rationale: Prevents tampering, enforces boot integrity

---

Exploit Mitigations

Memory Tagging Extension (MTE):
- Enabled (hardware-level memory safety)
- Detects use-after-free, buffer overflows

Pointer Authentication (PAC):
- Enabled (prevents ROP/JOP attacks)

Branch Target Identification (BTI):
- Enabled (CFI enforcement)

Hardened Malloc:
- GrapheneOS custom allocator
- Additional memory corruption protections

---

Profile Security

Profile Separation:
- Owner: Empty (system management)
- Main: Daily use (isolated)
- Banking: Financial (isolated)
- GPS: Play Services (sandboxed)

Per-Profile PINs:
- 8-digit random (no patterns)
- No biometrics (cannot be coerced)
- Separate for each profile

Auto-Lock:
- Screen timeout: 1 minute
- Profile lock: Immediate on screen off

---

Permissions (Default Deny)

Network:
- Off by default (all apps)
- On-demand grant (per-session)
- Revoked on app close

Sensors:
- Camera: Disabled by default
- Microphone: Disabled by default
- Location: GPS profile only (sandboxed)

Storage:
- Storage Scopes enforced
- Per-app isolated folders
- No shared storage access

Contacts:
- Denied by default
- Signal: Full access (exception)
- All others: Denied

---

Network Hardening

Connectivity Checks:
- Disabled (prevents Google DNS leaks)

Secure DNS:
- Private DNS: Mullvad DoH
- Fallback: Disabled

Captive Portal:
- Disabled (manual handling if needed)

---

System Security

App Installation:
- Unknown sources: Per-profile control
- Owner profile: never
- Main: via Obtainium
- Banking: Verified sources only
- GPS: Sandboxed Play (isolated)

Updates:
- Automatic security updates: Enabled
- Reboot for updates: Manual (controlled timing)

USB:
- Charging-only default
- Data transfer: Manual enable (temporary)

SIM PIN:
- Enabled (if cellular used)

---

Sandboxed Google Play

Profile: GPS only  
Isolation: Full (no privileged access)  
Network: VPN enforced

Capabilities:
- App installation from Play Store
- In-app purchases (if needed)
- Push notifications (FCM)

Limitations:
- No system-level access
- No cross-profile communication
- No Google account sync outside GPS profile

---

Backup Configuration

Native Backup:
- Enabled: Yes
- Destination: Homelab Server (local network)
- Encryption: Built-in (GrapheneOS)
- Frequency: Monthly + pre-major changes

Excluded:
- Owner profile (nothing to backup)
- GPS profile
- Temporary cache/data

---

Additional Hardening

Scramble PIN Layout:
- Enabled (randomizes keypad)

Auto-Reboot:
- Enabled (72 hours idle → BFU state)
- Rationale: Clears encryption keys from memory

Duress PIN/Password:
- Not configured (consider for high-risk scenarios)