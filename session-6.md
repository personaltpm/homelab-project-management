# Session 6: Backup Strategy Implementation & External Storage Integration
**Status:** ✅ Complete  
**Time Invested:** 2 hours 35 minutes (1 hour NFS setup + 20 min scheduling + 1 hour HA research/manual backup + 15 min SMB troubleshooting)

## Proxmox Backup Implementation:
**Configuration approach:** Derek Seaman tutorial for NFS setup, modified for GUI configuration
- Tutorial reference: [Synology NFS for Proxmox Backup Server Datastore](https://www.derekseaman.com/2025/08/how-to-synology-nfs-for-proxmox-backup-server-datastore-2025.html)
- Adaptation: Used Proxmox web interface instead of shell commands for storage configuration
- Target storage: Synology DS218j with NFS v3 protocol

## Synology NFS Configuration:
- Service enabled: Control Panel → File Services → NFS
- Shared folder created: `proxmox-bkup` with dedicated path `/volume1/proxmox-bkup`
- NFS permissions: Proxmox host IP (192.168.50.X) with read/write access
- Security setting: "No mapping" for proper root access from Proxmox
- Storage verification: NFS mount test successful via Proxmox web interface

## Proxmox Backup Automation:
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

## Backup Validation:
**Manual backup test results:**
- VM size: 32.0 GiB (Home Assistant VM)
- Compressed archive: 1.46 GB final size
- Transfer speed: 1.1 GiB/s sustained throughput
- Duration: Completed in under 30 minutes
- Status: Successful with no errors
- Storage verification: Backup file confirmed present on Synology NAS

## Home Assistant Backup Strategy:
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

## Key Technical Decisions:
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

## Backup Redundancy Strategy:
**External storage approach validated:**
- Primary backup: Proxmox VM backups to Synology NAS (weekly, 1.46 GB compressed)
- Secondary backup: HA configuration backups to Synology NAS (daily, ~3 MB via SMB)
- Tertiary redundancy: Synology cloud sync provides off-site protection
- Quaternary option: Manual HA exports for critical configuration preservation
- Single point of failure eliminated: Backup storage independent of host hardware

## Current Backup Architecture Status:
- **Proxmox Level:** Weekly automated VM backups operational
- **Home Assistant Level:** Daily automated configuration backups operational via network storage
- **Local Backup Capability:** Manual HA backup creation and download functional for immediate needs
- **Storage Verification:** Both backup types confirmed writing to appropriate Synology shares
- **Automation Health:** All automated backup processes operational with next execution scheduled

# Outstanding Challenges & Next Steps

## Network Architecture Limitation:
Consumer router cannot provide granular control for secure IoT integration. Permanent choice required between isolation and automation.

---

## Planned Session 7 Scope:
- Configure direct HomeKit integration (Ecobee thermostat)
- Implement Google Calendar integration for appointment-based triggers
- Begin automation migration from existing platforms (SmartThings, Google Home, IFTTT)
- Test and document advanced automation scenarios
- Evaluate migration strategy from cloud-dependent integrations to local control

## Future Integration Plans:
- Direct HomeKit integrations: Honeywell, August
- Advanced automation workflows combining calendar events with device triggers
- Migration timeline and rollback strategy for existing SmartThings automations
- Performance optimization and system monitoring setup

