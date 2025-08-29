# Project Progress Log

## Session 2: Network Configuration & Troubleshooting
**Status:** 🔄 In Progress - Reinstall Required  
**Time Invested:** 3 hours  
**Key Resolution:** Clean reinstall with ethernet connected from start

### Hardware Validation Results:
- **TP-Link UE300:** RTL8153 chipset recognized by Proxmox
- **USB 3.0 connectivity:** Adapter detected via `lsusb`
- **Network cable:** Confirmed working on other devices
- **Mesh network:** DHCP assignment working (192.168.50.x subnet)

### Issues Encountered & Troubleshooting Process:

#### Issue 1: Interface Naming Instability
**Problem:** USB ethernet adapter interface name changed between connections
- Initial: `enx[MAC-BASED-ID-1]`
- After reconnection: `enx[MAC-BASED-ID-2]`
- **Impact:** Static bridge configuration referenced non-existent interface

**Troubleshooting Steps:**
1. `lsusb | grep -i ethernet` - Confirmed adapter detection
2. `ip link show` - Identified actual interface name
3. `ip addr show` - Verified IP assignment but interface DOWN state
4. `ip link set enx[MAC-BASED-ID-2] up` - Manually brought interface up

#### Issue 2: Network Connectivity Failure
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

#### Issue 3: Configuration Complexity
**Root Cause:** Network configured without ethernet adapter present during initial Proxmox installation
- WiFi installation impossible (confirmed limitation)
- Bridge-to-interface mapping misconfigured
- Multiple interface naming conflicts accumulated

### Decision: Clean Reinstall Approach
**Rationale:** 
- Interface naming conflicts require extensive network config rebuilds
- Clean install with ethernet present eliminates configuration drift
- Time-efficient alternative to continued troubleshooting

**Validated Compatibility:**
- RTL8153 chipset works with Proxmox
- TP-Link UE300 hardware detection successful
- Network infrastructure confirmed functional

### Key Learnings:
1. **USB ethernet adapters must be connected during Proxmox installation** for proper interface detection
2. **RTL8153 chipset compatibility confirmed** - no driver issues encountered  
3. **Interface naming instability** is a real concern with USB adapters
4. **Systematic troubleshooting approach** validated each network layer
5. **Strategic decision-making:** Clean reinstall can be more efficient than debugging configuration drift

### Next Session Focus:
- [ ] Reinstall Proxmox with TP-Link UE300 connected from boot
- [ ] Configure network during installation process  
- [ ] Document clean installation experience
- [ ] Validate web interface access immediately post-install

---

**Session 3 Planning:**  
Fresh installation approach with hardware validation lessons applied.

---

## Session 1: 2025/08/25 - Proxmox Installation & Network Discovery
**Status:** 🔄 In Progress  
**Time Invested:** 2 hours

### Planned vs Actual:
| Task | Original Plan | Actual Status | Notes |
|------|---------------|---------------|-------|
| Backup Surface Book 2 data | Day 1 | ✅ Complete | |
| Windows license extraction | Day 1 | ✅ Complete | |
| Link MS account | Day 1 | ✅ Complete | |
| Document BIOS settings | Day 2 | ✅ Complete | |
| Create Proxmox USB installer | Day 2 | ✅ Complete | Used Etcher to flash ISO |
| Test Proxmox live boot | Day 3 | ✅ Complete | |
| Full Proxmox installation | Day 4 | 🔄 Partial | Install complete, network config pending |

### Key Discovery: WiFi Installation Blocker
- **Issue:** Proxmox installation cannot connect to WiFi during initial setup
- **Impact:** Unable to access web interface for configuration
- **Research:** Video tutorial used instead of written guides ([YouTube reference](https://www.youtube.com/watch?v=zngSuqCM4d8))
- **Solution:** USB ethernet adapter required for network connectivity

### Hardware Decision: Ethernet + USB Strategy
**Decision:** Start with budget ethernet adapter approach
- **Rationale:** Standard virtual networking (not passthrough) reduces compatibility requirements
- **Target:** USB 3.0 to Gigabit adapter with Realtek RTL8153 chipset (~$15-20)
- **Future scope:** Use existing USB hubs for Z-Wave dongle passthrough testing

### Current System State
- Proxmox installation: ✅ Complete
- Web interface access: ❌ Blocked by network
- Next blocker: Ethernet adapter procurement and network configuration

### Timeline Adjustment
**New approach:** Session-based tracking instead of daily milestones
- **Target:** ~6 hours/week (2 hours × 3 sessions)
- **Realistic range:** 4-8 hours/week depending on schedule

### Community Resources Used:
- [Proxmox Installation Video Tutorial](https://www.youtube.com/watch?v=zngSuqCM4d8) - More helpful than written guides

### Next Session Focus:
- [ ] Purchase USB ethernet adapter (Realtek RTL8153 chipset)
- [ ] Configure Proxmox networking via web interface
- [ ] Document network configuration for future reference
- [ ] Begin Home Assistant VM creation

### Key Learnings:
- WiFi during Proxmox installation is not practical/possible
- Video tutorials more effective than documentation for initial setup
- Hardware validation phase is necessary before proceeding
- Session-based timeline more realistic than daily milestones

### Network Architecture Clarification:
- **Ethernet:** Standard virtual networking through Proxmox bridge
- **USB passthrough:** Only needed for Z-Wave dongle (Zooz), not ethernet
- **Future devices:** Zigbee/Matter dongles will require passthrough research

---

**Session 2 Planning:**
Focus on network connectivity and basic Proxmox configuration once ethernet adapter arrives.

---
