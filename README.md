# Home Assistant Home Lab Project

## Project Overview

**What:** Migration from SmartThings to self-hosted Home Assistant for complete smart home control and automation

**Hardware:** Proxmox Virtual Environment on Surface Book 2 (8CPU/16GB RAM) running Home Assistant OS VM, Synology DS218j NAS for backups

**Current State:** Operational baseline with external access, 150+ entities integrated (ZWave/Zigbee via SmartThings cloud bridge + direct integrations), Google Assistant voice control, notification infrastructure across three systems (NAS/Proxmox/HA), comprehensive backup strategy (local NAS + offsite iDrive)

**Goal:** Fully local smart home automation with direct device control, eliminating cloud dependencies where possible, maintaining reliability and expanding automation capabilities beyond SmartThings limitations

## Documentation Structure

This repository documents the complete build process through sequential session files. Each session captures:
- Technical implementation details and architectural decisions
- Troubleshooting processes and root cause analysis
- Lessons learned (both technical and project management)
- Outstanding issues and workarounds

Sessions are grouped by project phase:
- **Infrastructure (S1-6):** Base system installation, networking, backup configuration
- **Configuration (S7-10):** Device integration, entity management, external access, voice automation
- **Migration (S11+):** Direct ZWave control, comprehensive automation implementation

The documentation prioritizes teaching value - detailed troubleshooting when solutions were non-obvious, compressed coverage when processes worked as expected. See individual session files for complete technical narratives.

## Session Index

### Infrastructure Phase (S1-6)

### [Session 1: Planning & Requirements](session-1.md)
Project scope definition, hardware assessment, phased timeline with realistic risk evaluation

### [Session 2: Proxmox Installation Failure](session-2.md)
WiFi installation impossible, USB ethernet requirement discovered, 3h debug abandoned for clean reinstall approach

### [Session 3: Proxmox Success](session-3.md)
Ethernet-connected boot enabled immediate network access, helper scripts deployed, power management configured

### [Session 4: Home Assistant VM Deployment](session-4.md)
Community script installation (2CPU/4GB RAM/32GB disk), auto-discovery validated eight integration types, hybrid SmartThings strategy established

### [Session 5: Device Integration & Network Challenges](session-5.md)
IoT isolation blocking auto-discovery, temporary removal enabled integration testing, consumer router limitations identified

### [Session 6: Backup Infrastructure](session-6.md)
Proxmox NFS backup to Synology (Mon 2AM, 5d/4w/2mo/1y retention), HA Samba Backup addon configured, critical router reboot test passed

### Configuration Phase (S7-10)

### [Session 7: Direct Integration Expansion](session-7.md)
HACS installed, Ecobee/August/Mysa HomeKit integration (physical reset resolved pairing), 66 entities renamed with caching issues, LG ThinQ and Fitbit OAuth configured

### [Session 8: System Organization & Dashboard Strategy](session-8.md)
Area model restructured (removed personal devices from location-based system), labels created but limited utility discovered, Mushroom framework installed, functional dashboard approach validated over categorical, Wyze integration added (motion detection persistence issue identified)

### [Session 9: Monitoring & ZWave Test Migration](session-9.md)
Telegram notification infrastructure operational (NAS/Proxmox/HA), ZWave firmware updated (7.17.1→7.18.3), 2 switches migrated ST→HA direct with full entity exposure, "Outside Lights On" automation 100% reliable after migration (ST cloud bridge command-dropping eliminated), Honeywell T5 OAuth replaced with HomeKit alternative

### [Session 10: External Access & Voice Automation Foundation](session-10.md)
NAS hardware replaced (6TB SHR), 3TB data restored via iDrive, Nabu Casa operational with Google Assistant integrated, voice automation architecture established via Scripts POC

### Migration Phase (S11+)

### Session 11: Full ZWave Migration
**In Progress: 33+ switches ST→HA direct, voice automation Script expansion, entity replacement workflow*

### Session 12: Notification Implementation
*Planned: Comprehensive HA notification strategy, integration monitoring, device offline alerts*

### Session 13: Dashboard Completion
*Planned: Status dashboard, functional dashboard expansion*

## Current Architecture

### Compute & Network
- **Proxmox Host:** 192.168.50.X, Surface Book 2 (8CPU/16GB RAM), sleep disabled, USB passthrough via Device ID
- **Home Assistant VM:** 192.168.50.X, 2CPU/6GB RAM/32GB disk, HAOS 16.2, DHCP reserved, externally accessible via Nabu Casa
- **Synology NAS:** 192.168.50.X, DS218j, 6TB single drive (SHR), NFS+Samba-Backup active, iDrive daily offsite backup, Drive 1 (4TB healthy) available for future redundancy
- **Router:** ASUS ZenWiFi (5 years old), IoT isolation disabled, Monday 3AM reboot schedule
- **External Access:** Nabu Casa cloud relay (active, validated)

### Integration Paths
- **ZWave:** 2 direct HA (Tenant Side Porch, Side Light) + 31 ST cloud bridge (pending S11 migration)
- **Zigbee:** ~10 devices via ST cloud bridge
- **WiFi Direct:** Kasa switches/plugs (local control), Wyze (HACS integration + Docker for cameras)
- **HomeKit Local:** 3 Ecobee thermostats, August lock, Mysa thermostat, Honeywell T5 (limited entity exposure)
- **Cloud Direct:** LG ThinQ (washer/dryer), Fitbit, Google Calendar
- **Voice Control:** Google Assistant via Nabu Casa (2 devices exposed, 3 Scripts operational: bedtime/sleep/morning)
- **Notifications:** Telegram (NAS + Proxmox + HA), test automations validated

### Backup Strategy
- **Proxmox:** Monday 2AM via NFS to Synology (retention: 5 daily/4 weekly/2 monthly/1 yearly)
- **Home Assistant:** Daily 2AM via Samba Backup addon to Synology (retention: 10 backups)
- **Offsite:** iDrive daily backup from Synology (3TB, validated restore capability)
- **Architecture:** Hardware isolation prevents cascade failures (NAS crashes validated as not destabilizing HA/Proxmox)

## Key Technical Decisions

**Infrastructure Choices:**
- **Proxmox over dedicated hardware:** Multi-VM capability, free, strong community, preserves Windows 10 OEM as VM option
- **USB ethernet over WiFi:** Kernel-level stability, interface naming consistency (validated: TP-Link UE300 RTL8153)
- **Helper scripts over manual config:** Community-Scripts repo for repeatable, validated deployments
- **NFS for Proxmox, SMB for HA:** Protocol-appropriate (NFS simpler for automated VM backups, SMB standard for HA addon)

**Integration Strategy:**
- **Hybrid ST bridge + direct:** Gradual migration preserves functionality, validates direct control benefits before full commitment
- **Cloud bridge limitations discovered:** ST drops commands in rapid sequence (5-device automation = 1 consistent failure), limited entity exposure (no dimmer levels, no advanced config)
- **Direct ZWave benefits:** Full entity exposure, OTA firmware capability, 100% automation reliability, advanced configuration access

**Platform & Architecture:**
- **Nabu Casa over DIY:** $65/year justified by security confidence + time savings vs DuckDNS/Let's Encrypt complexity, funds HA development, 31-day trial validated
- **Scripts for voice automation:** HA logic + Google voice interface separation enables maintainability during device migrations (HA shows unavailable entities vs Google auto-deletes), supports parameters (brightness_pct), allows complex conditions
- **SHR over Basic/RAID:** Single-drive start with future redundancy addition capability without data migration, learned from RAID-0 zero-redundancy risk
- **iDrive offsite over local-only:** RAID-0 failure taught importance of offsite backup, 3TB restore validated in 4 days

**Backup Architecture:**
- **Samba Backup addon over SMB mount:** Addon pushes TO NAS (safe), mount IN HA caused crashes during router reboots (S6 discovery)
- **Time-based retention over count-based:** Inclusive of setup phase learning, captures configuration evolution

## Critical Issues & Workarounds

**SmartThings Cloud Bridge Reliability:**
- **Issue:** Rapid-sequence commands drop consistently (5-device automation = 4 success, 1 failure, no error reported)
- **Root cause:** ST cloud bridge timing failures, network latency + command queuing
- **Impact:** "Outside Lights On" automation unreliable via ST
- **Resolution (S9):** Migrated 2 switches to direct ZWave, automation now 100% reliable (4/4 devices respond)
- **S11 plan:** Full migration of 33+ switches eliminates ST bridge dependency

**SmartThings Entity Exposure Limitations:**
- **Issue:** ST bridge exposes only basic on/off, hides advanced capabilities (dimmer levels, manufacturer metadata, firmware info, advanced config)
- **Impact:** Severely limits automation possibilities (no brightness checking, no conditional logic on dimmer state)
- **Resolution:** Direct ZWave integration provides full entity exposure + OTA firmware capability

**Google Home State Sync:**
- **Issue:** Manual control in app requires double-tap (first tap: light responds but Google UI reverts, second tap: syncs correctly)
- **Pattern:** Voice commands work perfectly, HA→Google sync works, only Google app manual control affected
- **Root cause:** State reporting race condition during initial pairing (Google expects immediate confirmation, Z-Wave mesh + HA response delayed)
- **Status:** Monitoring for self-resolution (typically stabilizes within 24-48 hours as cache establishes), voice control primary use case

**Wyze Motion Toggle Persistence:**
- **Issue:** HA sets motion detection off → Wyze app loads on → HA reverts setting
- **Possible causes:** Cloud API polling conflict, rate limiting, auth scope restrictions
- **Status:** Monitoring, motion detection functional for automations despite toggle state inconsistency

**Honeywell T5 Limited Entity Exposure:**
- **Issue:** HomeKit integration exposes temp/on-off/sensor only, no hold/schedule/advanced controls
- **Background:** OAuth developer approval pending months, no response
- **Resolution:** HomeKit via thermostat reset menu, acceptable for light use case
- **Future:** Replace thermostat if limitations become problematic (not current priority)

## Key Lessons

**Technical Insights:**
- USB ethernet DURING install (not after) prevents interface naming instability
- Physical device reset > software troubleshooting (Mysa: 7 attempts vs 1 reset successful)
- Consumer routers (5+ years) force isolation XOR automation tradeoff (newer models may handle granular rules better)
- Cloud bridges drop rapid commands (timing failures), direct control eliminates issue
- Dashboard strategy before building (functional > categorical) prevents wasted effort on wrong patterns
- Hardware architecture isolation prevents cascade failures (NAS crashes didn't destabilize HA, validated S6 design)
- Supergroup migration in Telegram not auto-handled (UI detected clearly with new chat IDs, YAML showed generic errors)
- Manufacturer-specific tools required for firmware updates (HA addons manage Z-Wave network, not USB stick hardware)
- USB passthrough: Device ID > port-based (survives physical hardware changes, firmware updates)
- S2 security QR code bypass (photograph during initial pairing to avoid future device removal for re-scanning)
- Direct ZWave provides full entity exposure + OTA firmware (ST bridge basic only, hidden advanced capabilities)
- Iterative device migration maintains functionality (test 2 switches before committing to 33+, validates benefits)
- Node-RED complexity not justified for simple notifications (HA native sufficient, defer unless complex logic needed)
- NFS service enabled ≠ running after hardware restart (explicit service restart required post-NAS drive swap)
- iDrive restore time-intensive (~4 days for 3TB) but reliable, incremental validation (folder-by-folder, twice) provides integrity confidence
- SHR with single drive = future-proof for redundancy addition without data migration (vs Basic requiring backup/restore for RAID conversion)
- Scripts vs Scenes: Scripts provide parameter control (brightness_pct explicit values) and logic capability vs static state snapshots requiring re-save on changes
- Typos in webhook URLs (botAPI- vs bot) cause silent failures, test messages critical for validation after config changes

**Project Management:**
- Session-flex framework: work-based not time-based post-infrastructure phase (S1-6 had dedicated blocks, S7+ incremental across days)
- Realistic time including research + failures (document actual investment, not just successful attempts)
- Document during not after (context loss risk, patterns emerge in real-time)
- Pragmatic > perfect (phased approach, test minimal scope first)
- Hardware validate before migrate (2 ZWave switches before 33+, NAS backup cycles before major work)
- Phased purchasing reduces financial risk (validate approach before full investment)
- Test minimal scope first (2 devices before 33, 3 Scripts before full voice automation migration)
- Functional > theoretical (simple solutions when both work, ask before over-engineering)
- Critical evaluation before work (question assumptions, challenge "building for nothing" scenarios)
- Platform evaluation requires threat model assessment (Telegram: security trade-offs explicit, E2E encryption vs convenience/bidirectional for use case)
- UI config can handle edge cases better than YAML (Telegram supergroup migration: UI showed exact new chat IDs, YAML generic errors only)
- Tool UX quality ≠ functionality (Simplicity Studio slow/unclear but successfully updated Z-Wave firmware)
- Real-world failure testing > synthetic tests (NAS crash during backup window validated Telegram notification + HA stability)
- Troubleshooting time can exceed implementation (Telegram YAML hours vs UI 10min, non-obvious solutions require patience)
- POC sufficient for infrastructure validation (defer comprehensive implementation when scope large, validate pattern first)
- Nabu Casa cost ($65/year) justified by time savings + security confidence vs DIY complexity (factor learning curve, ongoing maintenance, misconfiguration risk)
- Incremental restore (folder-by-folder, twice) vs bulk restore trades time for validation confidence (worth investment for 3TB critical data)
- POC with 3 Scripts before full migration validates architecture before committing to pattern (prevents rework if approach flawed)
- Deferring voice automation Script creation (for ZWave devices) to S11 minimizes rework (create once with correct entities vs create with ST, migrate, update to HA)
- Hardware changes during project require reconfiguration time (NAS drive swap: Telegram + NFS + Samba + iDrive re-setup, factor into timeline)
- Google Home device reference auto-deletion (when unpaired) vs HA unavailable entity retention makes HA superior for tracking dependencies during migrations

## Resources & Credits

**Installation & Configuration:**
- [Community Scripts for Proxmox](https://github.com/community-scripts/ProxmoxVE) - Helper scripts for HA VM deployment
- [Proxmox Installation Video](https://youtube.com/watch?v=zngSuqCM4d8) - Network configuration reference
- [Derek Seaman NFS Guide](https://derekseaman.com/2025/08/synology-nfs-proxmox) - Synology NFS backup setup (adapted for GUI)
- [SmartHomeJunkie Google Calendar](https://smarthomejunkie.net/how-to-use-calendar-events-in-home-assistant-tutorial/) - OAuth configuration (adapted for Google UI changes)
- [HACS Documentation](https://hacs.xyz/docs/use/) - Integration installation
- [Home Assistant Fitbit Integration](https://home-assistant.io/integrations/fitbit/) - OAuth setup when UI provided no link

**Notifications & Monitoring:**
- [Synology DSM 7.1 Telegram Alerts](https://medium.com/@nikolayburyak/synology-dsm-7-1-telegram-alerts-b9c27fcbf8de) - Webhook configuration
- [Proxmox Telegram Webhook Forum](https://forum.proxmox.com/threads/problem-with-webhook-notification-targets.162839/) - JSON POST body solution for URL encoding issues

**Firmware & Maintenance:**
- [Zooz ZST10 Firmware Update Guide](https://support.getzooz.com/kb/article/931-how-to-perform-an-ota-firmware-update-on-your-zst10-700-z-wave-stick/) - Simplicity Studio process (7.17.1→7.18.3)

**Hardware:**
- TP-Link UE300 USB Ethernet Adapter (RTL8153 chipset) - Kernel-stable, validated
- Zooz ZST10-700 Z-Wave USB stick - Firmware updates via manufacturer tools, not HA addons

## Project Status

**Current Phase:** Configuration baseline complete, external access operational, voice automation architecture validated

**Completed (S1-10):**
- Proxmox installation with stable networking (ethernet)
- Home Assistant VM deployment and base configuration
- Backup infrastructure (Proxmox NFS + HA Samba + iDrive offsite, all validated)
- Device integration: 150+ entities (WiFi direct, HomeKit local, ST cloud bridge, cloud services)
- Dashboard POC: functional approach validated
- Notification infrastructure: Telegram across three systems (NAS/Proxmox/HA)
- ZWave test migration: 2 switches direct, automation reliability validated
- NAS hardware replacement: 6TB SHR, 3TB restored via iDrive
- External access: Nabu Casa operational, Google Assistant integrated
- Voice automation: Scripts architecture validated (3 operational)

**In Progress:**
- Google Home state sync monitoring (expected self-resolution)
- Voice automation expansion planning (S11: Scripts for ZWave devices)

**Next Major Milestones:**
- **S11:** Full ZWave migration (33+ switches ST→HA direct), voice automation Script expansion, entity replacement workflow
- **S12:** Comprehensive HA notification strategy (integration monitoring, device offline alerts, automation failures, system health)
- **S13:** Dashboard completion (status dashboard, functional dashboard expansion)
- **Post-S13:** Operational baseline reached, future sessions driven by scope expansion or learning value

**Open Issues:**
- Full ZWave migration: S11 (33+ switches from ST bridge to direct HA control)
- Voice automation Script creation: S11 (automations using ZWave devices, create with ST entities, replace post-migration)
- HA notification implementation: S12 (comprehensive monitoring strategy)
- Dashboard expansion: S13 (status dashboard, additional functional dashboards)
- Wyze motion toggle persistence (monitor, not blocking)
- Honeywell T5 limited entity exposure (monitor adequacy, replace thermostat if limitations problematic)
- NAS redundancy: Add Drive 1 (4TB) post-S11 for SHR fault tolerance (deferred until after high-risk migration work)

**Critical Success Factors Validated:**
- Backup strategy comprehensive and tested (local + offsite, restore capability proven)
- External access secure and functional (Nabu Casa validated)
- Direct control benefits proven (ZWave reliability + entity exposure vs ST limitations)
- Voice automation architecture maintainable (Scripts survive device migrations, Google automations unchanged)
- Hardware isolation prevents cascade failures (NAS issues don't destabilize HA/Proxmox)

---

**STATUS:** S1-10-complete | External-access-operational (Nabu Casa) | Google-Assistant-integrated | Voice-automation-POC-validated | NAS-6TB-SHR-stable | 3TB-restored-iDrive-active | Ready-S11: full-ZWave-migration (33+ switches)

**Last Updated:** Session 10 completion, incremental configuration phase with external access and voice automation foundation established
