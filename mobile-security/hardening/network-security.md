Network Security

Last Updated: 2026-01-19

---

VPN Configuration

Provider: Mullvad VPN  
Payment: Crypto (anonymous)  
Protocol: WireGuard

Settings:
- Always-on: Enabled
- Block connections without VPN: Enabled (kill switch)
- Split-tunneling: Available (manual, rare use)

Per-Profile:
- Main: VPN enforced
- Banking: VPN enforced
- GPS: no VPN

---

DNS Configuration

Primary: Mullvad DoH (DNS-over-HTTPS)  
Fallback: Disabled  
Leak Prevention: Private DNS enforced

Rationale:
- Encrypted DNS queries
- No ISP/network DNS visibility
- Mullvad integration (consistent IP)

---

Network Permissions

Default: Denied (all apps)  
Grant Model: On-demand, per-session  
Revocation: on app close

Exception: Apps requiring persistent connectivity (e.g. Signal)

---

Wi-Fi Security

Auto-Connect: Disabled (manual selection)  
Saved Networks: Minimal (home, trusted only)  
Public Wi-Fi: VPN mandatory (kill switch active)

MAC Randomization:
- Enabled (per-network)
- Non-persistent MAC addresses

---

Cellular Security

SIM PIN: Enabled  
Data: VPN enforced  
SMS: Avoid (Signal preferred)

2G: Disabled (baseband setting, if available)  
Rationale: 2G = no encryption, IMSI catcher vulnerable

---

Connectivity Checks

Status: Disabled  
Rationale: Prevents Google DNS/connectivity probes

Trade-off: Manual captive portal handling (rare)

---

Sandboxed Play Network Isolation

Profile: GPS only  
VPN: Enforced (same as other profiles)  
Isolation: No cross-profile network access

FCM (Push Notifications):
- Routed through VPN
- Minimal metadata exposure

---

Threat Mitigations

MITM Attacks:
- VPN encryption (all traffic)
- Certificate pinning (where available)
- Private DNS (DoH)

ISP Surveillance:
- VPN hides traffic content
- DNS queries encrypted (DoH)
- IP address masked (Mullvad exit nodes)

Public Wi-Fi Risks:
- Kill switch prevents unencrypted fallback
- VPN mandatory before connectivity
-> don't use any public or untrusted/unknown wifi

---

Emergency Network Access

Scenario: VPN failure, urgent connectivity needed

Procedure:
1. Disable kill switch (Settings → VPN)
2. Enable network (aware of unencrypted risk)
3. Perform urgent task (minimal)
4. Re-enable kill switch immediately
5. Verify VPN reconnects

Use Cases: Emergency calls, critical communication (rare)