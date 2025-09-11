# Project Progress Log
---

## Session 6: Backup Strategy Implementation & External Storage Integration
**Status:** ✅ Complete  
**Time Invested:** 2 hours 35 minutes (1 hour NFS setup + 20 min scheduling + 1 hour HA research/manual backup + 15 min SMB troubleshooting)

### Proxmox Backup Implementation:
**Configuration approach:** Derek Seaman tutorial for NFS setup, modified for GUI configuration
- Tutorial reference: [Synology NFS for Proxmox Backup Server Datastore](https://www.derekseaman.com/2025/08/how-to-synology-nfs-for-proxmox-backup-server-datastore-2025.html)
- Adaptation: Used Proxmox web interface instead of shell commands for storage configuration
- Target storage: Synology DS218j with NFS v3 protocol

### Synology NFS Configuration:
- Service enabled: Control Panel → File Services → NFS
- Shared folder created: `proxmox-bkup` with dedicated path `/volume1/proxmox-bkup`
- NFS permissions: Proxmox host IP (192.168.50.X) with read/write access
- Security setting: "No mapping" for proper root access from Proxmox
- Storage verification: NFS mount test successful via Proxmox web interface

### Proxmox Backup Automation:
**Storage configuration:**
- ID: synology-backup
- Storage type: NFS
- Content type: VZDump backup files
- Server: 192.168.50.X (Synology IP)
- Export path: `/volume1/proxmox-bkup`

**Automated scheduling:**
- Frequency: Weekly (Monday 2:00 AM)
- Target VMs: All VMs (automatic inclusion of future VMs)
- Retention policy: Time-based (5 daily, 4 weekly, 2 monthly, 1 yearly)
- Compression: ZSTD for optimal size/speed balance

### Backup Validation:
**Manual backup test results:**
- VM size: 32.0 GiB (Home Assistant VM)
- Compressed archive: 1.46 GB final size
- Transfer speed: 1.1 GiB/s sustained throughput
- Duration: Completed in under 30 minutes
- Status: Successful with no errors
- Storage verification: Backup file confirmed present on Synology NAS

### Home Assistant Backup Strategy:
**Manual backup capability:** Successfully created and downloaded HA backup files
- Backup scope: Full system backup including configuration, add-ons, and data
- File size: Compact compared to full VM backup
- Manual process: Settings → System → Backups → Create backup

**Network storage implementation:** SMB connectivity to Synology NAS configured
- **Initial Challenge:** "Mounting HA_NAS did not succeed" error during SMB setup
- **Root Cause Analysis:** Share path format and SMB protocol version mismatch
- **Solutions Applied:**
  - Remote share path: Changed from `volume1\ha-bkups` to `ha-bkups` (SMB expects share name only)
  - CIFS version: Changed from "Auto (2.1+)" to "SMB2.0" (matched Synology maximum protocol)
- **Rationale for SMB Protocol:** HA add-on ecosystem has broader SMB/CIFS compatibility than NFS alternatives
- **Dedicated Share Strategy:** Separate `ha-bkups` folder isolates HA backups from Proxmox VM archives

**Automated backup validation:**
- Status: Fully operational with daily automated backups to network storage
- Test backup: 3.07 MB automatic backup created and stored successfully
- File verification: Backup confirmed in Synology `ha-bkups` folder
- Automation health: "2 automatic backups, 6.08 MB in total" with scheduled daily execution
- **Dual Backup Strategy Rationale:** Local manual capability preserved for immediate access, network storage provides automated off-site protection

### Key Technical Decisions:
**NFS vs SMB protocol selection:**
- Proxmox: NFS chosen for better Linux integration and performance
- Home Assistant: SMB chosen for broader HA add-on compatibility after protocol version alignment
- Dual protocol approach: Leverages optimal protocols for each system

**Retention strategy rationale:**
- Time-based over count-based: Provides recovery options across different time horizons
- Granular retention: Recent backups dense, historical backups sparse
- Setup phase approach: Intentionally inclusive retention during active development to preserve restore points
- Future optimization: Retention policies will be refined once system stabilizes and usage patterns are established
- Storage efficiency: Current settings balance protection with Synology capacity during learning phase

### Backup Redundancy Strategy:
**External storage approach validated:**
- Primary backup: Proxmox VM backups to Synology NAS (weekly, 1.46 GB compressed)
- Secondary backup: HA configuration backups to Synology NAS (daily, ~3 MB via SMB)
- Tertiary redundancy: Synology cloud sync provides off-site protection
- Quaternary option: Manual HA exports for critical configuration preservation
- Single point of failure eliminated: Backup storage independent of host hardware

### Current Backup Architecture Status:
- **Proxmox Level:** Weekly automated VM backups operational
- **Home Assistant Level:** Daily automated configuration backups operational via network storage
- **Local Backup Capability:** Manual HA backup creation and download functional for immediate needs
- **Storage Verification:** Both backup types confirmed writing to appropriate Synology shares
- **Automation Health:** All automated backup processes operational with next execution scheduled

## Outstanding Challenges & Next Steps

### Network Architecture Limitation:
Consumer router cannot provide granular control for secure IoT integration. Permanent choice required between isolation and automation.

---

### Planned Session 7 Scope:
- Configure direct HomeKit integration (Ecobee thermostat)
- Implement Google Calendar integration for appointment-based triggers
- Begin automation migration from existing platforms (SmartThings, Google Home, IFTTT)
- Test and document advanced automation scenarios
- Evaluate migration strategy from cloud-dependent integrations to local control

### Future Integration Plans:
- Direct HomeKit integrations: Honeywell, August
- Advanced automation workflows combining calendar events with device triggers
- Migration timeline and rollback strategy for existing SmartThings automations
- Performance optimization and system monitoring setup

### Documentation Strategy:
- GitHub for complete technical documentation
- Reddit for community sharing once advanced automation scenarios are complete
- Focus on realistic timelines and decision-making process for complex integrations

---

## Session 5: Device Integration & Network Architecture Implementation
**Status:** ✅ Complete  
**Time Invested:** 4 hours (including network troubleshooting and integration testing)  
**Key Achievement:** Functional home automation system with device control and validated backup strategy

### Network Challenge Resolution:
**Problem:** IoT devices isolated on separate network preventing Home Assistant discovery
- **Router:** ASUS ZenWiFi AX6600 Mini with IoT network isolation enabled
- **Impact:** HA could only discover hardwired devices, not WiFi-based automation devices

**Solution implemented:** Temporary IoT network isolation removal
- **Risk mitigation:** Time-limited exposure during active configuration
- **Validation:** Successful device discovery confirmed via ping testing
- **Architecture decision:** Accept temporary security tradeoff for automation functionality

### Device Integration Results:
**SmartThings Integration:**
- **Scope:** Existing Zigbee/Z-Wave devices maintained on SmartThings hub
- **Control method:** Cloud bridge integration with Home Assistant
- **Entity management:** Selective disabling of WiFi devices from SmartThings path in HA
- **Validation:** Switch control testing confirmed bidirectional communication

**Direct WiFi Integration:**
- **TP-Link Kasa devices:** Native integration successful
- **Control validation:** Dual-path testing confirmed both SmartThings and direct control
- **Architecture cleanup:** Removed duplicate entities by disabling SmartThings path for WiFi devices
- **Performance:** Direct integration faster than cloud routing

**Ecobee Integration:**
- **Challenge:** HomeKit code requirements encountered
- **Solution:** Utilized existing SmartThings integration path
- **Status:** Functional through cloud bridge, direct integration deferred

### Automation Functionality Testing:
- **Basic automation:** Switch-to-switch trigger/response testing successful
- **Response time:** Near-instantaneous device control confirmed
- **Entity visibility:** Device area assignment resolved automation device selection
- **Architecture validation:** Direct WiFi control operational for automation triggers

### Hardware Validation:
**Z-Wave USB Dongle Passthrough:**
- **Device:** Zooz Z-Wave dongle
- **Proxmox configuration:** USB device passthrough to Home Assistant VM successful
- **HA integration:** Z-Wave JS add-on installation and configuration completed
- **Status:** Hardware path validated for future device integration

### Backup Strategy Implementation:
**External storage approach:**
- **Target:** Synology NAS for network-attached backup storage
- **Redundancy:** NAS-to-cloud backup for additional protection
- **Rationale:** Avoid single point of failure with laptop hardware
- **Implementation:** Network share backup capability confirmed for future automation

### Network Architecture Final State:
| Component | Integration Path | Control Method | Status |
|-----------|-----------------|----------------|--------|
| Z-Wave devices | SmartThings → HA | Cloud bridge | Functional |
| Zigbee devices | SmartThings → HA | Cloud bridge | Functional |
| WiFi devices (Kasa) | Direct HA integration | Local network | Functional |
| Ecobee thermostat | SmartThings → HA | Cloud bridge | Functional |
| Future Z-Wave | Direct HA (USB dongle) | Local control | Hardware validated |

### Key Learnings:
1. **Consumer router limitations:** Granular firewall rules often unavailable for selective network bridging
2. **Integration architecture:** Hybrid approach balances security, performance, and complexity
3. **Entity management:** Selective integration disabling prevents duplicate device entities
4. **USB passthrough reliability:** Hardware abstraction works correctly for USB device integration
5. **Network isolation tradeoffs:** Security vs automation functionality requires conscious decisions
6. **IoT isolation incompatibility:** Asus Consumer router cannot support selective network access - choice between full isolation or full automation access required

### Technical Decisions Made:
- **IoT isolation temporarily removed:** Functionality prioritized over network segmentation
- **Direct WiFi integration preferred:** Local control chosen over cloud dependency where possible
- **SmartThings as radio bridge:** Maintained for devices requiring specific protocol hardware
- **External backup strategy:** Network storage selected over local backup

### Session Stopping Point:
- Complete device integration across multiple protocols
- Validated automation functionality with working triggers
- Hardware capabilities confirmed for future expansion
- Backup strategy planned with external storage redundancy

### Next Session Focus:
- [ ] Implement automated backup scheduling to Synology NAS
- [ ] Configure one direct HomeKit integration (Ecobee)
- [ ] Migrate 2-3 key automations from existing platforms to HA
- [ ] Test automation reliability and performance

### Resources and Credits:
- **Z-Wave JS Add-on:** Home Assistant community-maintained integration
- **TP-Link Kasa Integration:** Native Home Assistant integration
- **SmartThings Integration:** Home Assistant cloud-to-cloud bridge

---

**Session 6 Planning:**  
Advanced automation configuration and backup implementation with focus on system optimization and community documentation.

---

## Session 4: Home Assistant VM Deployment & Initial Setup
**Status:** ✅ Complete  
**Time Invested:** 1 hour total (45 min research/evaluation + 15 min successful deployment)  
**Key Achievement:** Home Assistant VM running with device discovery active

### Research, Decision Making & Strategic Planning:
- **Storage VM evaluation:** Determined unnecessary for single-host deployment
- **Installation method research:** Community script selected over manual VM configuration
- **Resource allocation:** Accepted script defaults based on community testing

**Integration Strategy Decision:**
**Hybrid approach selected for device management:**
- **SmartThings integration:** Existing Z-Wave/Zigbee devices remain on ST hub, controlled via HA cloud integration
- **Direct HA integration:** New devices and WiFi-based devices connect directly to HA
- **Future migration:** Manual device migration to direct HA control when time permits

**Rationale:**
- Avoids disrupting existing device pairings and Z-Wave mesh topology
- Enables immediate HA automation capabilities without re-pairing complexity
- Provides pathway for gradual migration to local control
- Reduces initial setup complexity while maintaining future flexibility

**USB dongle testing:** Z-Wave dongle passthrough validation planned as future-proofing measure, not immediate requirement

### VM Creation Process:
- **Method:** Community script deployment via Proxmox shell
- **Script used:** `bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/vm/haos-vm.sh)"`
- **VM specifications:** 2 CPUs, 4GB RAM, 32GB disk (script defaults)
- **Network configuration:** Automatic DHCP assignment (192.168.50.X)
- **Installation time:** ~15 minutes for VM creation and HA OS boot

### Home Assistant Initial Setup:
- **Web interface access:** Successful at `http://192.168.50.X:8123`
- **Account creation:** Initial user setup completed
- **Device discovery:** Automatic detection of network devices during onboarding
- **Discovered devices:** Google Cast, Internet Printing Protocol, Obihai, Reolink, SmartThings, UPnP/IGD

### Technical Validation:
- **VM networking:** Proper bridge connectivity confirmed (separate IP from host)
- **Web interface responsiveness:** Smooth operation from remote browser
- **Device discovery functionality:** Successfully identifying actual network devices
- **System stability:** VM running independently of management laptop

### Network Architecture Confirmed:
| Component | IP Address | Status |
|-----------|------------|--------|
| Proxmox host | 192.168.50.X | Stable |
| Home Assistant VM | 192.168.50.X | Running |
| Management access | Any network device | Functional |

### Integration Readiness Assessment:
**Devices available for integration:**
- **Reolink cameras:** Network-accessible, ready for configuration
- **SmartThings hub:** Detected, integration available for existing Z-Wave/Zigbee devices
- **Google Cast devices:** Discovered, media control possible
- **WiFi devices:** Direct integration planned

### Resources and Credits:
- **Home Assistant VM script:** [Community Scripts for ProxmoxVE](https://github.com/community-scripts/ProxmoxVE)
- **Script maintainers:** Community-driven project providing Proxmox automation scripts
- **Installation method reference:** Community script approach reduces manual configuration complexity

### Session Stopping Point:
- Home Assistant web interface accessible and stable
- Device discovery completed successfully
- Account creation and basic setup finished
- Integration strategy defined for next phase

### Key Learnings:
1. **Community scripts significantly reduce configuration complexity**
2. **Automatic device discovery works effectively on mesh networks**
3. **VM resource allocation handled appropriately by established scripts**
4. **Network segmentation working correctly** (VM has independent IP)
5. **Hybrid integration strategy balances complexity vs functionality**

### Next Session Focus:
- [ ] Configure SmartThings integration for existing Z-Wave/Zigbee devices
- [ ] Integrate WiFi-based devices (Reolink cameras, network devices)
- [ ] Test Z-Wave USB dongle passthrough for future validation
- [ ] Set up basic automations for validation
- [ ] Test device control functionality

---

**Session 5 Planning:**  
Device integration execution with focus on SmartThings bridge setup and direct WiFi device connections.

---

## Session 3: Successful Clean Installation & Post-Installation Optimization
**Status:** ✅ Complete  
**Time Invested:** 1.5 hours (45 min install + 30 min optimization)  
**Key Achievement:** Proxmox fully operational with optimized configuration

### Clean Installation Results:
- **Installation approach:** Ethernet adapter connected during entire process
- **Network detection:** USB ethernet adapter (`enx[MAC-BASED-ID]`) recognized immediately
- **Bridge configuration:** `vmbr0` automatically configured with DHCP (192.168.50.50/24)
- **Web interface access:** Successful on first attempt from remote laptop
- **Post-reboot validation:** Network configuration stable after restart

### Post-Installation Optimization:
- **Tutorial reference:** YouTube video optimization guide (15:50-end)
- **Scripts executed:** Post-installation helper scripts for repository and performance optimization
- **Repository configuration:** Enterprise repository issues resolved
- **System updates:** Latest packages and security patches applied
- **Reboot test:** System maintained network stability and interface naming

### Validation of Hardware Compatibility:
- **TP-Link UE300:** RTL8153 chipset fully supported with stable interface naming
- **Surface Book 2:** UEFI installation successful with graphical installer
- **USB 3.0 connectivity:** Interface maintained consistency across reboot cycles
- **Network infrastructure:** Router properly recognizes Proxmox bridge IP

### Installation Process:
1. **Pre-installation:** USB ethernet adapter connected to Surface Book 2
2. **UEFI boot:** Volume Down + Power method successful
3. **Graphical installer:** Network auto-detected during installation
4. **Bridge configuration:** Automatic setup with proper interface binding
5. **Post-install optimization:** Helper scripts and system updates
6. **Reboot validation:** Network configuration persistence confirmed

### Network Configuration Comparison:
| Metric | Session 2 (Failed) | Session 3 (Success) |
|--------|-------------------|---------------------|
| Installation method | WiFi attempted | Ethernet connected from start |
| Interface detection | Post-install, naming conflicts | During install, stable naming |
| Bridge configuration | Manual troubleshooting required | Automatic configuration |
| Connectivity | 3+ hours debugging, no resolution | Immediate access |
| Post-reboot stability | Not tested | Validated working |
| Total time | 3+ hours | 1.5 hours |

### Post-Installation Configuration:
- **Power management:** Sleep/suspend disabled for 24/7 operation
- **Repository optimization:** Enterprise repository conflicts resolved
- **System updates:** Current packages and security patches installed
- **Interface stability:** Ethernet adapter maintained configuration across reboot

### Key Learnings Applied:
1. **Ethernet connection during installation** eliminated all interface naming and bridge configuration issues
2. **Post-installation optimization** resolved repository and update issues
3. **Reboot testing** confirmed network configuration persistence
4. **Clean installation approach** proved more efficient than troubleshooting

### Current System State:
- **Proxmox Version:** 9.0.3 (optimized)
- **Network Interface:** Stable across reboot cycles
- **Bridge:** `vmbr0` persistent configuration
- **Power:** Sleep/suspend disabled for continuous operation
- **Access:** Remote web interface fully functional post-optimization

### Next Session Focus:
- [ ] Create Home Assistant VM
- [ ] Configure VM networking and resource allocation  
- [ ] Install Home Assistant Operating System
- [ ] Basic HA web interface validation
- [ ] Plan backup strategy for post-configuration

### Success Metrics Met:
- ✅ Proxmox installation complete and accessible
- ✅ Post-installation optimization completed
- ✅ Network configuration stable across reboots
- ✅ Repository and update issues resolved
- ✅ Ready for VM deployment phase

---

**Session 4 Planning:**  
Home Assistant VM creation with validated stable Proxmox foundation.

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
