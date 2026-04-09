# Network Isolation

Network segmentation strategy using Qubes OS proxy VMs and firewall rules.

## Network Architecture
```
Internet
  ↓
sys-net (HVM, PCI passthrough)
  ↓
sys-firewall (ProxyVM)
  ↓
sys-mullvad (ProxyVM, WireGuard VPN)
  ↓
├── dev
├── learning
├── browser
└── sys-ssh

Airgapped (netvm: "")
├── sys-git
├── sys-pgp
├── sys-ssh-agent
├── sys-usb
└── vault
```

## Network Qubes

**sys-net:**
- Type: HVM
- Purpose: Direct hardware access (PCI device passthrough)
- Network: Direct internet via physical NIC
- Provides network to: sys-firewall

**sys-firewall:**
- Type: ProxyVM
- Purpose: Network filtering, DNS enforcement
- Network: sys-net (upstream)
- Provides network to: sys-mullvad
- DNS: Forwards to Mullvad DNS (10.64.0.1)

**sys-mullvad:**
- Type: ProxyVM
- Purpose: WireGuard VPN endpoint
- Network: sys-firewall (upstream)
- Provides network to: All internet-connected AppVMs
- DNS: Mullvad DNS (10.64.0.1)
- VPN: WireGuard (auto-start via rc.local)

## Firewall Rules

**sys-mullvad (restrictive):**
```bash
# Allow DNS to Mullvad
iptables -I FORWARD -d 10.64.0.1 -p udp --dport 53 -j ACCEPT

# Allow VPN endpoint
iptables -I FORWARD -d <VPN_IP> -p udp --dport 51820 -j ACCEPT

# Allow established connections
iptables -I FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# Block everything else
iptables -P FORWARD DROP
```

DNS redirection:
```bash
iptables -t nat -A PR-QBS -p udp --dport 53 -j DNAT --to-destination 10.64.0.1
iptables -t nat -A PR-QBS -p tcp --dport 53 -j DNAT --to-destination 10.64.0.1
```

NAT for VPN:
```bash
iptables -t nat -A POSTROUTING -o wireguard -j MASQUERADE
```

## VPN Configuration

**WireGuard (sys-mullvad):**

Config: `/rw/config/wireguard/wireguard.conf.j2`
```ini
[Interface]
PrivateKey = {{ private_key }}
Address = {{ vpn_ip }}/32
DNS = 10.64.0.1

[Peer]
PublicKey = {{ server_public_key }}
Endpoint = {{ server_endpoint }}:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```

Auto-start: `/rw/config/rc.local`
```bash
#!/bin/bash
wg-quick up wireguard
```

**VPN killswitch:**
- Firewall rules enforce VPN-only traffic
- If WireGuard down → no internet access
- No DNS leaks (forced to 10.64.0.1)

## DNS Configuration

**sys-mullvad DNS:**
- Primary: 10.64.0.1 (Mullvad DNS)
- No fallback DNS (enforced by firewall)

**sys-firewall DNS redirection:**
- All downstream DNS queries → 10.64.0.1
- Prevents DNS leaks to ISP

**AppVM DNS:**
- Inherits from sys-mullvad (10.64.0.1)
- Transparent to applications

## Network Isolation Levels

**Level 1 - Airgapped:**
- sys-git (Git server, Qrexec-only)
- sys-pgp (GPG server, Qrexec-only)
- sys-ssh-agent (SSH agent server, Qrexec-only)
- sys-usb (USB devices)
- vault (offline credential storage)

**Level 2 - VPN-only:**
- dev (development)
- learning (documents)
- browser (web browsing)
- sys-ssh (SSH to external servers)
- dvm-mgmt (template updates)

## Qrexec-based Network Services

**sys-ssh (Qrexec TCP proxy):**

Service: `ConnectTCP+host+port`

Client config (`~/.ssh/config`):
```
Host example
    HostName localhost
    Port 1840
    ProxyCommand qrexec-client-vm -- sys-ssh ConnectTCP+example.com+22
```

Server-side (sys-ssh):
- Receives connection request via Qrexec
- Establishes TCP connection to target host
- Proxies traffic via sys-mullvad (VPN)

Policy:
```
ConnectTCP+* * @anyvm sys-ssh ask default_target=sys-ssh
```

**Advantages:**
- Client qube doesn't need network access
- SSH traffic always via VPN
- Centralized known_hosts management

## Network-less Services

**Git (sys-git):**
- Protocol: Qrexec (GitFetch, GitPush, GitInit)
- Network: None (airgapped)
- Client: `git clone qrexec://@default/repo-name`

**GPG (sys-pgp):**
- Protocol: Qrexec (qubes.Gpg2)
- Network: None (airgapped)
- Client: split-gpg2 integration

**SSH Agent (sys-ssh-agent):**
- Protocol: Qrexec (SshAgent+AGENT)
- Network: None (airgapped)
- Client: Socket forwarding to `/tmp/ssh-agent-forwarder/AGENT.sock`

## Network Traffic Flow Examples

**Web browsing (browser):**
```
browser → sys-mullvad (WireGuard) → sys-firewall → sys-net → Internet
              ↓
         10.64.0.1 (Mullvad DNS)
```

**Git push (dev):**
```
dev → Qrexec (GitPush) → sys-git (airgapped)
  No network traffic
```

**SSH to external server (dev):**
```
dev → Qrexec (Ssh) → sys-ssh → sys-ssh-agent (Qrexec)
                              ↓
                        sys-mullvad (VPN) → Internet
```

**Template update:**
```
tpl-dev → dvm-mgmt → sys-mullvad (VPN) → Internet
              ↓
        qubes-updates-proxy (port 8082)
```

## Update Proxy

**service.qubes-updates-proxy:**
- Enabled on: sys-firewall, sys-mullvad
- Purpose: HTTP proxy for template updates
- Port: 8082

**Update flow:**
1. Template starts update (apt/dnf)
2. Traffic → dvm-mgmt (default update proxy VM)
3. dvm-mgmt → sys-mullvad → VPN
4. Traffic returns via VPN → template

## Security Considerations

**VPN trust:**
- Mullvad provider: Trusted (paid with crypto, no logs)
- VPN compromise: Affects all internet traffic

**DNS privacy:**
- All DNS queries → Mullvad DNS (10.64.0.1)
- No ISP DNS exposure

**Network qube compromise:**
- sys-net/sys-firewall/sys-mullvad compromise: Full network visibility
- Mitigation: Minimal attack surface, disposable variants available

**Airgapped qube isolation:**
- sys-git: Cannot leak data over network
- sys-pgp: GPG keys never leave qube
- sys-ssh-agent: SSH keys never leave qube
- vault: Credentials never transmitted

## Disposable Network Qubes

**disp-sys-net:**
- Purpose: Disposable network qube (untrusted networks)
- Use case: Public Wi-Fi

**disp-sys-firewall:**
- Purpose: Disposable firewall
- Use case: Additional isolation layer

**Not currently used by default** (persistent preferred for stability)