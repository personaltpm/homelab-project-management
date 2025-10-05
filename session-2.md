# Session 2: Network Configuration & Troubleshooting
**Status:** 🔄 In Progress - Reinstall Required  
**Time Invested:** 3 hours  
**Key Resolution:** Clean reinstall with ethernet connected from start

## Hardware Validation Results:
- **TP-Link UE300:** RTL8153 chipset recognized by Proxmox
- **USB 3.0 connectivity:** Adapter detected via `lsusb`
- **Network cable:** Confirmed working on other devices
- **Mesh network:** DHCP assignment working (192.168.50.x subnet)

## Issues Encountered & Troubleshooting Process:

## Issue 1: Interface Naming Instability
**Problem:** USB ethernet adapter interface name changed between connections
- Initial: `enx[MAC-BASED-ID-1]`
- After reconnection: `enx[MAC-BASED-ID-2]`
- **Impact:** Static bridge configuration referenced non-existent interface

**Troubleshooting Steps:**
1. `lsusb | grep -i ethernet` - Confirmed adapter detection
2. `ip link show` - Identified actual interface name
3. `ip addr show` - Verified IP assignment but interface DOWN state
4. `ip link set enx[MAC-BASED-ID-2] up` - Manually brought interface up

## Issue 2: Network Connectivity Failure
**Problem:** Ping failures and web interface inaccessible despite correct IP configuration
- Both devices on same subnet (192.168.50.x)
- Intermittent ping responses (50% packet loss, then 0%, then total failure)
- Web service confirmed running locally via `curl -k https://localhost:8006`

**Troubleshooting Steps:**
1. **Firewall Investigation:**
   - `systemctl status pveproxy` - Confirmed web service running
   - `systemctl stop pve-firewall` - Eliminated firewall as cause
   - `ss -tulpn | grep 8006` - Verified port 8006 listening

2. **Bridge Configuration Analysis:**
   - `brctl show vmbr0` - Checked bridge membership  
   - `ip route show` - Verified routing table
   - Bridge had correct IP but physical interface wasn't properly joined

3. **Bidirectional Connectivity Tests:**
   - Laptop → Proxmox: Intermittent ping responses
   - Proxmox → Laptop: Total failure (destination unreachable)
   - Indicates asymmetric routing or bridge forwarding issue

## Issue 3: Configuration Complexity
**Root Cause:** Network configured without ethernet adapter present during initial Proxmox installation
- WiFi installation impossible (confirmed limitation)
- Bridge-to-interface mapping misconfigured
- Multiple interface naming conflicts accumulated

## Decision: Clean Reinstall Approach
**Rationale:** 
- Interface naming conflicts require extensive network config rebuilds
- Clean install with ethernet present eliminates configuration drift
- Time-efficient alternative to continued troubleshooting

**Validated Compatibility:**
- RTL8153 chipset works with Proxmox
- TP-Link UE300 hardware detection successful
- Network infrastructure confirmed functional

## Key Learnings:
1. **USB ethernet adapters must be connected during Proxmox installation** for proper interface detection
2. **RTL8153 chipset compatibility confirmed** - no driver issues encountered  
3. **Interface naming instability** is a real concern with USB adapters
4. **Systematic troubleshooting approach** validated each network layer
5. **Strategic decision-making:** Clean reinstall can be more efficient than debugging configuration drift

## Next Session Focus:
- [ ] Reinstall Proxmox with TP-Link UE300 connected from boot
- [ ] Configure network during installation process  
- [ ] Document clean installation experience
- [ ] Validate web interface access immediately post-install

---

**Session 3 Planning:**  
Fresh installation approach with hardware validation lessons applied.

