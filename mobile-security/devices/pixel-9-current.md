Pixel 9 - Current Configuration

Status: Production (Primary Device)  
OS: GrapheneOS (official support since Pixel 9 release)  
Role: All-in-one (temporary until OEM device)

---

Hardware

Model: Google Pixel 9  
Chipset: Tensor G4  
Security: Titan M2 security chip  
Bootloader: Locked (GrapheneOS verified boot)

---

Profile Configuration

Owner Profile
State: Empty  
Purpose: System management only

Main Profile
Purpose: Daily use & communications  
PIN: 8-digit random  
Apps: Signal, NewPipe, Infinity, KeePassDX, Aegis  
Network: Mullvad VPN always-on

Banking Profile
Purpose: Financial operations  
PIN: 8-digit random (separate)  
Apps: Bank apps, investment platforms, crypto wallets  
Network: VPN enforced, isolated

GPS Profile
Purpose: Google Play Services isolation  
PIN: 8-digit random (separate)  
Apps: Sandboxed Play + dependent apps  
Network: VPN enforced

---

Hardening

- Verified boot (custom AVB keys)
- Exploit mitigations (MTE, PAC, BTI)
- Network permissions: Off by default, on-demand
- Sensors: Disabled by default
- Storage Scopes: Enforced
- No biometrics (PIN-only)

---

Backup Strategy

GrapheneOS Native Backup:
- Destination: Homelab Server
- Frequency: Monthly or on major changes
- Encryption: GrapheneOS built-in

App Data:
- KeePass: Manual export to Proton Drive
- Aegis: Local backup (encrypted)
- 2FA codes: Paper + encrypted digital