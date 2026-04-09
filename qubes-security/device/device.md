#Lenovo T480

Lenovo ThinkPad T480 with Libreboot, Qubes OS, and hardware modifications.

##Hardware

**Base system:**
- Model: Lenovo ThinkPad T480
- CPU: Intel Core i5-8350U (4 cores, 8 threads)
- RAM: 32GB DDR4
- Storage: 1TB NVMe SSD
- Display: 14" 1920x1080 IPS

**Hardware modifications:**
- Camera: Removed (physical hardware removal)
- Microphone: Removed (physical hardware removal)

##Firmware

**Libreboot:**
- BIOS: Libreboot (replaced proprietary Lenovo BIOS)
- Boot: Coreboot-based, no Intel ME (Management Engine)
- Verification: Measured boot, TPM support

**Benefits:**
- No proprietary firmware blobs
- No Intel ME backdoor
- Full control over boot process
- Open-source firmware

##Boot Security

**Qubes Anti Evil Maid (AEM):**
- Purpose: Detect firmware tampering, Evil Maid attacks
- Method: TPM-based sealed secrets
- Boot verification: Checks firmware + bootloader integrity

**How it works:**
1. TPM seals secret during trusted boot
2. On next boot, TPM releases secret only if firmware unchanged
3. If firmware tampered → TPM refuses to unseal → AEM fails
4. User knows system compromised, does not enter disk password

**Setup:**
- TPM 1.2/2.0 enabled in Libreboot
- AEM configured during Qubes installation
- Secret stored in TPM, released only on valid boot

##Disk Encryption

**LUKS full-disk encryption:**
- All partitions encrypted (except /boot)
- Passphrase required on boot
- AEM verifies system integrity before password prompt

**Workflow:**
1. Power on → Libreboot
2. AEM checks firmware integrity via TPM
3. If AEM passes → LUKS password prompt
4. If AEM fails → Do not enter password (system compromised)
5. Enter LUKS password → Qubes OS boots

##Physical Security

**Camera removal:**
- Integrated webcam physically removed from motherboard
- No software-based camera disable (hardware gone)
- Rationale: No camera = no camera exploits

**Microphone removal:**
- Integrated microphone physically removed from motherboard
- No software-based microphone disable (hardware gone)
- Rationale: No microphone = no audio surveillance

**External peripherals:**
- USB webcam: Used when needed (plugged into sys-usb, isolated)
- USB microphone/headset: Used when needed (plugged into sys-usb, isolated)
- Benefit: Full control over when camera/mic available

##Security Features

**Libreboot:**
- No proprietary firmware
- No Intel ME (neutered/removed)
- No backdoors in boot process
- Open-source, auditable

**AEM:**
- Detects firmware tampering
- Detects bootloader tampering
- Protects against Evil Maid attacks
- TPM-based verification

**Hardware modifications:**
- No camera (physical removal)
- No microphone (physical removal)
- Reduces attack surface (no audio/video exploits)

##Threat Model

**Protected against:**
- Firmware tampering (AEM + TPM)
- Evil Maid attacks (AEM detects changes)
- Camera exploits (no camera)
- Microphone exploits (no microphone)
- Intel ME backdoors (Libreboot removes ME)

**Not protected against:**
- Physical access after boot (disk encryption only protects powered-off state)
- Hardware keyloggers (external USB devices)
- DMA attacks (Thunderbolt, FireWire - mitigated by sys-usb isolation)
- Supply chain attacks on other components (CPU, RAM, etc.)

##Maintenance

**Libreboot updates:**
- Manual process (flash new Libreboot image)
- Rare (only on critical security updates)
- Requires hardware programmer or internal flashing

**AEM re-sealing:**
- After Libreboot update → Re-seal TPM secret
- After kernel update → Re-seal TPM secret
- Procedure: Boot with AEM re-seal option

**LUKS password change:**
```bash
#In dom0 (after boot)
sudo cryptsetup luksChangeKey /dev/nvme0n1p3
```

##Backup Strategy

**Full system backup:**
- Qubes backup: All critical qubes (vault, dev, learning)
- Frequency: After significant changes
- Storage: External encrypted drive

**Firmware backup:**
- Libreboot image backed up externally
- TPM state documented (in case re-seal needed)

**Recovery:**
- Restore Qubes backup to new installation
- Re-flash Libreboot if hardware failure
- Re-seal AEM after firmware restore

##Performance

**Specs sufficient for:**
- 10+ concurrent qubes
- Development workloads (VSCodium, .NET, Node.js)
- Browsing (Brave, Mullvad Browser)
- Document editing (OnlyOffice)

**Memory allocation:**
- Dom0: 4GB reserved
- sys-net/sys-firewall/sys-mullvad: 300MB each
- dev/browser/learning: 4GB each
- Other qubes: 200-400MB each

**CPU:**
- 4 cores sufficient for most workloads
- VPN encryption: Minimal overhead
- Template updates: Parallel possible

##Known Issues

**Libreboot limitations:**
- No hardware video acceleration (Intel GPU firmware removed)
- Slightly slower boot compared to proprietary BIOS
- No suspend-to-RAM (S3 sleep) - use shutdown or hibernate

**Hardware modifications:**
- No integrated camera/mic (intentional, not a bug)
- External USB required for video calls

**Workarounds:**
- Video acceleration: Not needed for typical use
- Sleep: Use hibernate or shutdown
- Camera/mic: USB devices when needed (sys-usb isolation)

##Future Upgrades

**Considered:**
- Heads firmware (alternative to Libreboot, better AEM integration)
- Hardware kill switches (physical toggle for Wi-Fi, Bluetooth)
- Battery replacement (original aging)

**Not planned:**
- Newer hardware (T480 sufficient, Libreboot support critical)
- Re-adding camera/mic (security > convenience)