# Home Assistant Migration Project

Personal home automation infrastructure built on Proxmox VE + Home Assistant, documenting the systematic migration from SmartThings/IFTTT to local control with hybrid cloud integration.

## Project Overview

**Planning Documentation:** [Original Project Plan](PROJECT-PLAN.md) (timeline and risk assessment prior to execution)
**Hardware:** Surface Book 2 (8 CPU/16GB RAM) running Proxmox VE  
**Core System:** Home Assistant OS VM (2 CPU, 6GB RAM, 32GB disk)  
**Current State:** 148+ entities, hybrid SmartThings/direct integration architecture  
**Goal:** Centralized home automation with local control priority, gradual cloud service migration

## Documentation Structure

Sessions are documented chronologically with realistic timelines, decision rationale, and technical troubleshooting details. Focus on TPM/engineering case study approach rather than tutorial format.

---

## Infrastructure Foundation (Sessions 1-6)

### [Session 1: Planning & Research](Session-1.md)
Strategic approach, hardware selection rationale, success criteria definition

### [Session 2: Proxmox Installation - Failed Attempt](Session-2.md)
WiFi installation impossible, USB ethernet requirement discovered, 3+ hours troubleshooting

### [Session 3: Clean Proxmox Installation](Session-3.md)
Fresh start with ethernet from boot, 1.5h successful deployment, interface stability validated

### [Session 4: Home Assistant VM Deployment](Session-4.md)
Community script deployment, hybrid integration strategy decision, initial device discovery

### [Session 5: Device Integration & Network Challenges](Session-5.md)
IoT isolation vs automation tradeoff, direct integrations expanded, ZWave USB passthrough validated

### [Session 6: Backup Strategy Implementation](Session-6.md)
Proxmox → Synology NFS automated, HA → Samba Backup addon, SMB mount issues troubleshot

---

## System Organization & Expansion (Sessions 7-8)

### [Session 7: Direct Integration Expansion + Entity Management](Session-7.md)
Additional HomeKit/cloud integrations, entity rename marathon, caching issues discovery

### [Session 8: System Organization + Dashboard Architecture](Session-8.md)
Dashboard strategy validated, SmartThings entity limitations discovered, system maintenance events

---

## Monitoring & Device Migration (Sessions 9+)

### [Session 9: Monitoring & Notifications + ZWave Test]
*In Progress*

---

## Current Architecture

**Compute:**
- Proxmox VE on Surface Book 2 (8 CPU/16GB RAM)
- Home Assistant OS VM (2 CPU/6GB RAM/32GB disk)
- Synology DS218j NAS (backup target, RAID 0 + cloud sync)

**Network:**
- ASUS ZenWiFi router (5 years old, IoT isolation disabled for discovery phase)
- Wired: Proxmox host, HA VM, SmartThings hub, NAS
- Wireless: IoT devices (cameras, sensors, switches via SmartThings/direct)

**Integration Paths:**
- 35+ ZWave devices via SmartThings cloud bridge
- 10+ Zigbee devices via SmartThings cloud bridge  
- WiFi devices direct to HA (local control)
- HomeKit devices direct (3 Ecobee thermostats, August lock, Mysa thermostat)
- Cloud APIs: LG ThinQ, Fitbit, Google Calendar, Wyze

---

## Key Technical Decisions

- **Surface Book 2 over Raspberry Pi:** Stability concerns from community feedback, existing hardware with compute/storage headroom
- **USB Ethernet Adapter Required:** WiFi Proxmox installation impractical, ethernet must be connected during install for interface naming stability
- **Hybrid Integration Strategy:** Keep SmartThings for existing ZWave/Zigbee mesh (avoid re-pairing 35+ devices), direct integration for new devices, gradual migration approach
- **Backup Push Model:** HA pushes to NAS via Samba Backup addon vs mounting NAS storage in HA (avoids network dependency failure cascades)
- **Functional Dashboard Architecture:** Purpose-driven dashboards (housekeeper, cameras) vs categorical organization (reduces navigation overhead)

---

## Critical Issues & Workarounds

**SmartThings Cloud Bridge Reliability:**
- Commands dropped in rapid sequence (5 sent → 4 processed, 1 consistently missing)
- Timing-dependent failures (same device works manually, fails in automation)
- Workaround: Running automation from ST app temporarily, HA-native solution pending ZWave migration test

**SmartThings Entity Exposure Limitations:**
- Only basic on/off controls exposed to HA
- Missing: dimmer levels, manufacturer info, firmware version, advanced controls
- Impact: Cannot create brightness-based automations, building blind to actual device capabilities

**Wyze Motion Detection Toggle:**
- State doesn't persist when set via HA
- Workaround: IFTTT handles functionality temporarily (planned migration before canceling IFTTT subscription)

---

## Key Lessons

- **USB ethernet during install, not after:** Proxmox interface naming stability depends on hardware connected at installation time (3h debugging vs 1.5h clean install)
- **Physical device reset faster than software troubleshooting:** Mysa thermostat paired after physical reset (hold button) following 7 failed software integration attempts
- **Consumer router limitations:** Older routers (5+ years) force choice between IoT isolation and automation - granular firewall rules require prosumer/enterprise gear. Newer consumer routers may handle this better.
- **Cloud bridges drop rapid commands:** SmartThings consistently fails 1 of 5 simultaneous device commands, timing-dependent not device-specific
- **Dashboard strategy before building:** Validating functional > categorical approach early prevented wasted implementation time
- **Hardware architecture isolation prevents cascades:** NAS drive failures don't destabilize HA when network storage dependencies avoided

---

## Resources & Credits

- [Proxmox Community Scripts](https://github.com/community-scripts/ProxmoxVE)
- [HACS Documentation](https://hacs.xyz/docs/use/)
- Hardware: TP-Link UE300 USB Ethernet Adapter (RTL8153 chipset, Proxmox compatible)

---

**Project Status:** Active development, systematic phased approach, realistic timeline documentation for portfolio/build-public demonstration
