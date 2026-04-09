App Management

Last Updated: 2026-01-19

---

Installation Sources

Primary: Obtainium
Purpose: Centralized update management from trusted sources  
Sources:
- GitHub releases
- F-Droid repositories
- Direct APKs from developer websites

Advantages:
- No Google Play dependency
- Direct from source (supply chain reduction)
- Single-app update control
- Signature verification

Configuration:
- Auto-update: Disabled (manual review)
- Update notifications: Enabled
- Source verification: Always

Fallback: Aurora Store
Purpose: Access Play Store apps (anonymously)  
Usage: Avoided when possible  
Rationale: Obtainium preferred, Aurora for Play-exclusive apps

-> Aurora easier to compromise but no google account needed

Risk: in that case, privacy over security (due to minimal risk)

Configuration:
- Anonymous session (no Google account)
- VPN enforced
- Minimal usage

Sandboxed Google Play
Profile: GPS and Banking only
Purpose: Apps requiring Google Play Services  
Isolation: Full sandboxing (no privileged access)

Use Case: Apps with hard GMS dependency (banking apps, some services)

---

App Distribution

Main Profile
- Signal (full contact access)
- frontends (e.g. for yt, ...)
- KeePassDX (local password database)
- Aegis (2FA, local)
- [Other daily apps]

Banking Profile
- Bank apps (via sandboxed Play if required)
- Investment platforms
- Crypto wallets (Bitcoin, Monero, Bitpanda, etc.)

GPS Profile
- Sandboxed Google Play Services
- Apps requiring GMS (isolated)

---

## Update Strategy

Process:
1. Obtainium notifies of update vor regular manual check
2. Review changelog (GitHub/F-Droid)
3. Check for security issues or breaking changes
4. Manual update (per-app)
5. Verify app functionality post-update

Frequency: Weekly check, critical updates immediate

GrapheneOS Updates:
- Automatic security patches: Enabled
- Reboot: Manual (controlled timing)

---

App Vetting

Before Installation:
- [ ] Source verified (official GitHub, F-Droid, developer site)
- [ ] Permissions reviewed (minimal necessary)
- [ ] Open-source preferred (auditable)
- [ ] Community reputation checked
- [ ] Privacy policy reviewed (if applicable)

Red Flags:
- Excessive permissions (network + storage + sensors without justification)
- Closed-source with sensitive data access
- Unknown developer/unsigned APKs
- Google Play exclusive (no alternative source)

---

Permission Management

Default State: All permissions denied

Grant Process:
1. App requests permission
2. Evaluate necessity (least privilege)
3. Grant temporarily (per-session) if possible
4. Revoke after use (if not persistent need)

Exceptions:
- Signal: Full contact access (communication need)
- Banking apps: Network + storage (scoped)

---

App Isolation

Storage Scopes:
- Enforced per-app
- Dedicated folders only
- No shared storage access

Network:
- Per-app firewall (GrapheneOS)
- On-demand only
- Revoked on close (if temporary)

Sensors:
- Camera/mic: Disabled by default
- Location: GPS profile only

---

Backup Strategy

GrapheneOS Native:
- App data included (encrypted)
- Frequency: Monthly + pre-major changes

Manual Exports:
- KeePassDX: Database export to Proton Drive
- Aegis: Encrypted backup (local + Proton Drive)

---

Removal Policy

When to Remove:
- No longer needed
- Abandoned/unmaintained
- Privacy concerns emerge
- Excessive permission creep
- Replaced by better alternative

Process:
1. Export critical data (if applicable)
2. Revoke all permissions
3. Uninstall app
4. Verify data removed (Storage Scopes)