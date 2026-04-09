GrapheneOS OEM Device - Planned

Status: Tracking (Expected Q4 2026 / Q1 2027)  
Role: Future daily driver

---

Background

GrapheneOS is partnering with a major Android OEM to bring official support to flagship Snapdragon devices.

---

Why This Device

Non-Google Hardware:
- Ideological consistency (avoid Google ecosystem)
- Privacy-focused hardware choice

Performance:
- Snapdragon > Tensor G4
- Better power efficiency ?
- Superior sustained performance ?

Security:
- Official GrapheneOS support maintained
- Same security guarantees as Pixel devices
- Verified boot with custom keys

---

Planned Configuration

Profiles:
- Main (daily use, Signal, apps)
- GPS (sandboxed Play Services)

Role: Daily driver (24/7 usage)  
Exposure: High  
Risk Profile: Medium (contained by profile isolation)

---

Migration Plan

Phase 1: OEM device arrives  
Phase 2: Set up GrapheneOS + profiles  
Phase 3: Migrate Main profile data from Pixel 9  
Phase 4: Migrate GPS profile  
Phase 5: Pixel 9 becomes banking vault

Data Migration:
- GrapheneOS native backup restore
- KeePass/Aegis manual import
- Signal re-registration (new device)
- self hosted cloud
