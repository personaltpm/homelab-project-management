# homelab-project-management

📊 **[View Progress Log](PROGRESS.md)** | 🚧 **Status:** Week 1 In Progress

## Project Scope
Deploy a self-hosted home automation system using Proxmox as virtualization platform, running Home Assistant to manage smart home devices, with focus on security, reliability, and scalability for future additions. Secondary VM planned for Reolink/Wyze camera display system.

## Success Criteria
- Proxmox running stable on repurposed laptop
- Home Assistant VM successfully deployed and accessible via web interface
- At least 3 different device types integrated (e.g., lights, thermostat, sensors)
- Automated backup solution configured
- Local network access secured with HTTPS
- System maintains 99% uptime over 30-day period
- Windows 10 license preserved for future VM use

## Milestone Timeline
- Week 1: Research hardware requirements, backup laptop data, install Proxmox
- Week 2: Configure Proxmox networking, create Home Assistant VM
- Week 3: Initial Home Assistant setup, integrate first devices
- Week 4: Add remaining devices, configure automations
- Week 5: Implement backup strategy, security hardening
- Week 6: Documentation, performance optimization, lessons learn

## Hardware Requirements
| Component | Current State | Future Need | Priority |
|-----------|---------------|-------------|----------|
| Z-Wave | ✅ USB dongle owned | - | Complete |
| Zigbee | ❌ Not owned | USB dongle required | High - Week 3 |
| Matter | ❌ Not owned | USB dongle (optional) | Low - Month 2 |
| USB Expansion | ✅ Hub available | - | Complete |
| Network | WiFi (built-in) | USB ethernet adapter | Medium - Week 2 |

## Risk Register
| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Surface Book/Proxmox compatibility | Medium | High | Test with live USB first, document workarounds |
| Windows license won't transfer to VM | High | Low | Document phone activation method, MS account linking |
| Laptop hardware insufficient | Medium | High | Test with live USB first, have Plan B hardware |
| Network conflicts with existing router | Low | Medium | Document current network, plan separate VLAN |
| WiFi instability for automation | Medium | Medium | Plan ethernet adapter purchase after validation |
| Learning curve for Proxmox | High | Low | Allow extra time, identify Discord/Reddit communities |

## Decisions Log
**2025-07: Hardware Platform Selection**
- **Decision:** Use Surface Book 2 laptop instead of Raspberry Pi for Home Assistant
- **Rationale:** Community feedback indicates Pi has stability issues with HA; existing Surface Book 2 available at no cost
- **Trade-offs:** Surface may have compatibility issues with Proxmox, but web GUI management makes peripheral issues acceptable

**2025-07: Virtualization Platform - Proxmox VE**
- **Decision:** Selected Proxmox as hypervisor
- **Rationale:** Zero licensing cost, proven HA compatibility based on community deployments, allows future VM expansion
- **Future scope:** Second VM planned for Reolink/Wyze camera display system

**2025-07: Network Architecture**
- **Decision:** Start with WiFi, migrate to wired later
- **Rationale:** Prove feasibility before purchasing USB network adapter
- **Success criteria:** If stable on WiFi for 7 days, purchase dedicated ethernet adapter

**2025-07: Windows 10 License Preservation Strategy**
- **Decision:** Keep Windows 10 (not upgrade to 11), link to Microsoft account before Proxmox installation
- **Rationale:** Digital license ties to MS account, improves reactivation chances in VM
- **Risk:** No guaranteed method for OEM license transfer to VM
- **Approach:** Document multiple extraction methods, include phone activation as fallback

## Hardware Requirements

| Component | Current State | Future Need | Priority |
|-----------|---------------|-------------|----------|
| Z-Wave | ✅ USB dongle owned | - | Complete |
| Zigbee | ❌ Not owned | USB dongle required | High - Week 3 |
| Matter | ❌ Not owned | USB dongle (optional) | Low - Month 2 |
| USB Expansion | ✅ Hub available | - | Complete |
| Network | WiFi (built-in) | USB ethernet adapter | Medium - Week 2 |

## Pre-Installation Checklist

### Windows License Preservation
```powershell
# 1. Get current Windows edition
Get-ComputerInfo | select WindowsProductName, WindowsVersion

# 2. Get activation status
Get-CimInstance SoftwareLicensingProduct -Filter "Name like 'Windows%'" | where licensestatus -eq 1 | select name, licensestatus

# 3. Extract product key (may show BBBBB for digital)
wmic path softwarelicensingservice get OA3xOriginalProductKey
(Get-WmiObject -query 'select * from SoftwareLicensingService').OA3xOriginalProductKey

# 4. Save activation details
slmgr /dli > C:\activation_backup.txt

# 5. Document hardware UUID
wmic csproduct get UUID
### Microsoft Account Linking
- [ ] Settings → Accounts → Sign in with Microsoft account
- [ ] Verify: Settings → Update & Security → Activation
- [ ] Should show "Windows is activated with a digital license linked to your Microsoft account"
- [ ] Screenshot activation status

### BIOS/UEFI Documentation
```powershell
# Check current settings from Windows
bcdedit | findstr /i "path"
echo $env:firmware_type  # Shows Legacy or UEFI
Confirm-SecureBootUEFI   # Get Secure Boot status
Get-Tpm                  # Document TPM status
```
**Enter BIOS** (Surface Book 2: Hold Volume Up + Power):
- [ ] Document current boot mode (UEFI/Legacy)
- [ ] Document Secure Boot status (will disable for Proxmox)
- [ ] Document TPM status (leave enabled)
- [ ] Verify Virtualization enabled (VT-x/AMD-V) - CRITICAL
- [ ] Check VT-d status (for USB passthrough)
- [ ] Photo all BIOS screens before changes

### Required BIOS Changes for Proxmox
- Secure Boot: MUST disable
- Virtualization (VT-x): MUST enable
- VT-d: Enable if available (for USB passthrough to HA)

### Additional Backup Tasks
- [ ] Backup personal files
- [ ] Save activation files: `C:\Windows\System32\spp\store\2.0\`
- [ ] Run ShowKeyPlus/ProduKey to extract embedded BIOS key
- [ ] Note Surface Book 2 serial number (for phone activation)

## Windows VM Activation Plan (Priority Order)

1. **Digital License via MS Account**
  - Sign into same MS account in VM
  - Settings → Activation → Troubleshoot
  - Select "I changed hardware on this device recently"

2. **Phone Activation** (Good fallback)
  ```cmd
slui 4  # Opens phone activation wizard
```
- Select country → Get installation ID
- Call Microsoft → Say "hardware failure, reinstalling"
- Enter confirmation ID

3. **Manual Key Entry**
```cmd
slmgr /ipk [YOUR-PRODUCT-KEY]
slmgr /ato
```

4. **Activation Troubleshooter**
```cmd
slmgr /rearm
# Restart then retry activation
```

## Lessons Learned
[To be updated throughout project]

