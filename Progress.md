# Project Progress Log

---
## Session 3: Successful Clean Installation
**Status:** ✅ Complete  
**Time Invested:** 45 minutes  
**Key Achievement:** Proxmox fully operational with network access

### Clean Installation Results:
- **Installation approach:** Ethernet adapter connected during entire process
- **Network detection:** USB ethernet adapter (`enx[MAC-BASED-ID]`) recognized immediately
- **Bridge configuration:** `vmbr0` automatically configured with DHCP (192.168.50.50/24)
- **Web interface access:** Successful on first attempt from remote laptop
- **Power management:** Sleep/suspend disabled to prevent VM interruptions

### Validation of Hardware Compatibility:
- **TP-Link UE300:** RTL8153 chipset fully supported
- **Surface Book 2:** UEFI installation successful with graphical installer
- **USB 3.0 connectivity:** Stable interface detection and naming
- **Network infrastructure:** Mesh network DHCP assignment working correctly

### Installation Process:
1. **Pre-installation:** USB ethernet adapter connected to Surface Book 2
2. **UEFI boot:** Volume Down + Power method successful
3. **Graphical installer:** Network auto-detected during installation
4. **Bridge configuration:** Automatic setup with proper interface binding
5. **First boot:** Immediate web interface accessibility

### Network Configuration Comparison:
| Metric | Session 2 (Failed) | Session 3 (Success) |
|--------|-------------------|---------------------|
| Installation method | WiFi attempted | Ethernet connected from start |
| Interface detection | Post-install, naming conflicts | During install, stable naming |
| Bridge configuration | Manual troubleshooting required | Automatic configuration |
| Connectivity | 3+ hours debugging, no resolution | Immediate access |
| Total time | 3+ hours | 45 minutes |

### Post-Installation Configuration:
- **Power management:** `systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target`
- **Interface stability:** Ethernet adapter maintained same USB port throughout process
- **System validation:** Web interface, shell access, and VM creation capabilities confirmed

### Key Learnings Applied:
1. **Ethernet connection during installation** eliminated all interface naming and bridge configuration issues
2. **Clean installation approach** proved more efficient than extensive troubleshooting
3. **Systematic validation** confirmed all hardware components working as expected
4. **Power management configuration** prevents system sleep and VM interruptions

### Current System State:
- **Proxmox Version:** 9.0.3
- **Network Interface:** `enx[MAC-BASED-ID]` (USB ethernet stable)
- **Bridge:** `vmbr0` active with IP assignment
- **Power:** Sleep/suspend disabled for 24/7 operation
- **Access:** Remote web interface fully functional

### Next Session Focus:
- [ ] Create Home Assistant VM
- [ ] Configure VM networking and resource allocation
- [ ] Install Home Assistant Operating System
- [ ] Basic HA web interface validation
- [ ] Plan backup strategy for post-configuration

### Success Metrics Met:
- ✅ Proxmox installation complete and accessible
- ✅ Network configuration stable and functional  
- ✅ Power management configured for continuous operation
- ✅ Hardware compatibility validated
- ✅ Ready for VM deployment phase

---

**Session 4 Planning:**  
Home Assistant VM creation and initial OS installation with focus on network connectivity validation.

---

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
