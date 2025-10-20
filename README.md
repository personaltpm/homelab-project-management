# Home Assistant Migration Project

Personal home automation infrastructure built on Proxmox VE + Home Assistant, documenting the systematic migration from SmartThings/IFTTT to local control with hybrid cloud integration.

## Project Overview

**Hardware:** Surface Book 2 (8 CPU/16GB RAM) running Proxmox VE  
**Core System:** Home Assistant OS VM (2 CPU, 6GB RAM, 32GB disk)  
**Current State:** 150+ entities, hybrid SmartThings/direct integration architecture, 2 ZWave switches migrated directly  
**Goal:** Centralized home automation with local control priority, gradual cloud service migration  
**Planning Documentation:** [Original Project Plan](project-plan.md) (timeline and risk assessment prior to execution)

## Documentation Structure

Sessions are documented chronologically with realistic timelines, decision rationale, and technical troubleshooting details. Focus on TPM/engineering case study approach rather than tutorial format.

---

## Infrastructure Foundation (Sessions 1-6)

### [Session 1: Planning & Research](session-1.md)
Strategic approach, hardware selection rationale, success criteria definition

### [Session 2: Proxmox Installation - Failed Attempt](session-2.md)
WiFi installation impossible, USB ethernet requirement discovered, 3+ hours troubleshooting

### [Session 3: Clean Proxmox Installation](session-3.md)
Fresh start with ethernet from boot, 1.5h successful deployment, interface stability validated

### [Session 4: Home Assistant VM Deployment](session-4.md)
Community script deployment, hybrid integration strategy decision, initial device discovery

### [Session 5: Device Integration & Network Challenges](session-5.md)
IoT isolation vs automation tradeoff, direct integrations expanded, ZWave USB passthrough validated

### [Session 6: Backup Strategy Implementation](session-6.md)
Proxmox → Synology NFS automated, HA → Samba Backup addon, SMB mount issues troubleshot

---

## System Organization & Expansion (Sessions 7-8)

### [Session 7: Direct Integration Expansion + Entity Management](session-7.md)
Additional HomeKit/cloud integrations, entity rename marathon, caching issues discovery

### [Session 8: System Organization + Dashboard Architecture](session-8.md)
Dashboard strategy validated, SmartThings entity limitations discovered, system maintenance events

---

## Monitoring & Device Migration (Sessions 9+)

### [Session 9: Monitoring & Notifications + ZWave Migration Test](session-9.md)
Telegram notification infrastructure operational (NAS, Proxmox, HA), 2 ZWave switches migrated with improved control and OTA firmware capability, automation reliability validated

### Session 10: External Access + Google Assistant + NAS Replacement
*In Progress*

---

## Current Architecture

**Compute:**
- Proxmox VE on Surface Book 2 (8 CPU/16GB RAM)
- Home Assistant OS VM (2 CPU/6GB RAM/32GB disk)
- Synology DS218j NAS (backup target, RAID 0 with failing Drive 2, replacement pending)

**Network:**
- ASUS ZenWiFi router (5 years old, IoT isolation disabled for discovery phase)
- Wired: Proxmox host, HA VM, SmartThings hub, NAS
- Wireless: IoT devices (cameras, sensors, switches via SmartThings/direct)

**Integration Paths:**
- 33+ ZWave devices via SmartThings cloud bridge (2 migrated to direct HA control)
- 10+ Zigbee devices via SmartThings cloud bridge  
- WiFi devices direct to HA (local control)
- HomeKit devices direct (3 Ecobee thermostats, August lock, Mysa thermostat, Honeywell T5)
- Cloud APIs: LG ThinQ, Fitbit, Google Calendar, Wyze
- Notifications: Telegram (NAS, Proxmox, HA)

---

## Key Technical Decisions

- **Surface Book 2 over Raspberry Pi:** Stability concerns from community feedback, existing hardware with compute/storage headroom
- **USB Ethernet Adapter Required:** WiFi Proxmox installation impractical, ethernet must be connected during install for interface naming stability
- **Hybrid Integration Strategy:** Keep SmartThings for existing ZWave/Zigbee mesh (avoid re-pairing 35+ devices initially), direct integration for new devices, gradual migration approach validated via 2-switch test
- **Backup Push Model:** HA pushes to NAS via Samba Backup addon vs mounting NAS storage in HA (avoids network dependency failure cascades)
- **Functional Dashboard Architecture:** Purpose-driven dashboards (housekeeper, cameras) vs categorical organization (reduces navigation overhead)
- **Telegram for Notifications:** Bidirectional capability and multi-use platform outweigh E2E encryption concerns for home automation use case
- **USB Passthrough by Device ID:** More stable than port-based for ZWave stick (survives physical port changes)

---

## Critical Issues & Workarounds

**SmartThings Cloud Bridge Reliability:**
- Commands dropped in rapid sequence (5 sent → 4 processed, 1 consistently missing)
- Timing-dependent failures (same device works manually, fails in automation)
- **Resolution via ZWave migration:** 2 switches migrated to direct HA control, automation now 100% reliable (5/5 devices respond)
- Remaining 33+ switches still via ST bridge (full migration pending S11)

**SmartThings Entity Exposure Limitations:**
- Only basic on/off controls exposed to HA
- Missing: dimmer levels, manufacturer info, firmware version, advanced controls
- Impact: Cannot create brightness-based automations, building blind to actual device capabilities
- **Mitigation:** Direct ZWave migration provides full entity exposure (validated with 2 switches)

**Wyze Motion Detection Toggle:**
- State doesn't persist when set via HA
- Workaround: IFTTT handles functionality temporarily (planned migration before canceling IFTTT subscription)

**Synology NAS Drive 2 Failure:**
- RAID 0 with failing drive, periodic crashes (~weekly)
- Workaround: System restart restores functionality temporarily
- **Resolution pending:** S10 drive replacement and data migration

---

## Key Lessons

- **USB ethernet during install, not after:** Proxmox interface naming stability depends on hardware connected at installation time (3h debugging vs 1.5h clean install)
- **Physical device reset faster than software troubleshooting:** Mysa thermostat paired after physical reset (hold button) following 7 failed software integration attempts
- **Consumer router limitations:** Older routers (5+ years) force choice between IoT isolation and automation - granular firewall rules require prosumer/enterprise gear. Newer consumer routers may handle this better.
- **Cloud bridges drop rapid commands:** SmartThings consistently fails 1 of 5 simultaneous device commands, timing-dependent not device-specific. Direct ZWave eliminates issue.
- **Dashboard strategy before building:** Validating functional > categorical approach early prevented wasted implementation time
- **Hardware architecture isolation prevents cascades:** NAS drive failures don't destabilize HA when network storage dependencies avoided
- **Supergroup migration not handled automatically in Telegram config:** UI integration detected issue and clearly indicated chat ID changes (vs YAML generic errors)
- **Manufacturer-specific tools required for firmware updates:** HA addons manage ZWave network but can't update stick hardware
- **Iterative device migration maintains functionality:** Test per-device approach vs big bang cutover reduces risk

---

## Resources & Credits

- [Proxmox Community Scripts](https://github.com/community-scripts/ProxmoxVE)
- [HACS Documentation](https://hacs.xyz/docs/use/)
- [Synology DSM 7.1 Telegram Guide](https://medium.com/@nikolayburyak/synology-dsm-7-1-telegram-alerts-b9c27fcbf8de)
- [Proxmox Telegram Webhook Configuration](https://forum.proxmox.com/threads/problem-with-webhook-notification-targets.162839/)
- [Zooz ZST10 Firmware Update Guide](https://www.support.getzooz.com/kb/article/931-how-to-perform-an-ota-firmware-update-on-your-zst10-700-z-wave-stick/)
- Hardware: TP-Link UE300 USB Ethernet Adapter (RTL8153 chipset, Proxmox compatible)

---

**Project Status:** Active development, systematic phased approach, realistic timeline documentation for portfolio/build-public demonstration
