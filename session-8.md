# SESSION 8: System Organization + Dashboard Architecture (incremental timeline)
**Status:** ✅ Complete (with documented deferrals)  
**Time:** Incremental work across multiple days - entity management (hours due to caching), integration troubleshooting (Mysa 7 attempts), system maintenance  
**Achievement:** 148 entities systematically organized with strategic naming, dashboard architecture strategy validated (functional > categorical), critical system monitoring tests passed, integration expansion complete

---

## HACS + Framework Installation

**HACS:** Installed via hacs.xyz/docs/use/, provides custom integration repository access

**Mushroom Cards:** Installed via HACS, framework ready for dashboard building, complex implementation deferred pending ZWave device architecture decisions

---

## Integration Expansion

**Ecobee Thermostats (3 total via HomeKit):**
- **Method choice:** Direct cloud integration available but HomeKit chosen for local control. Possible fewer control options than cloud API but meets current needs, local priority.
- **Android limitation:** HomeKit pairing codes only display on physical thermostat screens, not in Ecobee app
- iPad workaround failed: only adds to Apple Home, doesn't display pairing codes for HA
- Resolution: 2 thermostats direct access, 1 required tenant coordination for code → all 3 operational
- **Entity advantage:** HomeKit integration provides full control (mode/hold/scheduling) vs SmartThings integration (sensor data only - temp/humidity)

**August Lock:** HomeKit direct integration, immediate success

**Honeywell T5:** 
- HA integration requires developer account, OAuth approval process
- Status: Submitted 3+ weeks ago, still pending approval

**Google Calendar:**
- Tutorial source: SmartHomeJunkie (outdated - Google OAuth changes, not HA changes)
- Adaptation required: manual navigation of Google developer console vs tutorial's automated flow
- Status: Configured and validated, weekend event automation confirmed working

**LG ThinQ (washer/dryer):**
- Method: Direct HA integration via LG ThinQ platform
- Setup: Device discovery → personal access token generation required (manual LG developer portal)
- Status: Operational

**Fitbit:**
- Method: Direct HA integration, OAuth required
- Setup challenge: Integration UI provided no setup link → manual navigation to HA docs (home-assistant.io/integrations/fitbit)
- OAuth: Manual token generation via Fitbit developer console
- **Area assignment challenge:** Personal wearable device doesn't fit location-based area model
- Initial workaround: Created "General" area (later removed during area model restructuring)

**Mysa bb-v2-0-l Thermostat (HomeKit) - 7 Failed Attempts:**
1. Auto-discovery hung when confirmation clicked (no pairing code prompt)
2. Manual integration attempt blocked by existing auto-discovery session
3. Integration restart via Developer Tools attempted (HomeKit not in reload dropdown)
4. Network discovery disable explored (no clear option in HA UI)
5. Full HA restart considered (user preference: targeted solution over system-wide restart)
6. Network troubleshooting (verified same SSID, IoT isolation already disabled in S5)
7. Developer account approach researched (HA has no native Mysa integration, only HomeKit path)

**SUCCESS (Attempt 8):** Physical device reset (hold down arrow button until device restarts) → auto-discovery immediately prompted for pairing code → paired successfully first attempt

**Root cause:** Mysa was in partial HomeKit pairing state preventing new pairing attempts. Physical reset cleared device-side HomeKit configuration.

**Known issue:** Mysa bb-v2-0-l model has documented problematic HomeKit implementation with third-party controllers - works reliably with Apple devices, fails cryptographic handshake with HA and similar platforms

---

## Entity Management Marathon

**Scope:** 66 total entity renames across 33 SmartThings devices + 33 direct integrations

**Process:**
- SmartThings devices: Re-enabled all devices/entities → used "Recreate Entity IDs" command with prefix → freed original names for direct integration renaming
- Direct integrations: Removed "2" suffixes from entity names, cleaned duplicate naming patterns
- Added device type suffixes for clarity ("_light", "_plug", "_switch")

**HA Caching Issue:**
- Entity name updates displayed in UI but reverted on page refresh
- Multiple browser windows/tabs open may have contributed to caching issues
- Workaround: Systematic refresh between each device update to force cache clear, still doing manual refreshes to be safe
- Process took hours due to required constant verification after each rename

**UI Automation Resilience:**
- Entity renames didn't break existing UI-based automations (HA auto-updated entity references)
- YAML automations would have broken (manual entity ID updates required)
- Validates UI automation approach for user workflow

---

## Dashboard POC Experiments

**Housekeeper Dashboard:**
- Person-specific approach
- Includes: Washer/dryer status + controls, door lock with planned time/day restrictions (not yet implemented)
- Validates: Person-centric dashboard model feasible

**Camera Dashboard:**
- Function-specific approach  
- All camera feeds consolidated
- Validates: Functional grouping vs location-based grouping

**Dashboard Architecture Strategy Evolution:**

*Rejected Approaches:*
- Category dashboards (cameras/status/sensors/etc) → Navigation overhead, too many dashboard switches
- Single main household dashboard → Too large, unwieldy at scale
- Personal dashboard (Fitbit/scale/health tracking) → Deferred, lower priority than household automation

*Accepted Approaches:*
- Functional dashboards for specific users or purposes (housekeeper, cameras)
- Manual device curation over bulk auto-add (intentional selection vs automatic inclusion)

**Key Discovery:** HA label-based filtering doesn't work dynamically for dashboards - labels less useful than anticipated for dashboard organization. Labels remain useful for device/entity list organization only.

**Outstanding Dashboard Work:**
- Status dashboard - deferred pending ZWave migration strategy results (see SmartThings Entity Exposure Limitations below)
- Additional functional-specific dashboards - framework validated, specific implementations pending
- Health/personal tracking dashboard - explicitly deferred (lower priority)

---

## Area Model Restructuring

**Problem:** Personal devices (Fitbit, scale) don't fit location-based area organization model

**Solution:**
- Removed area assignments from non-location devices and entities
- Clean separation: Location-based devices (lights/switches by room) vs function-based devices (personal tracking)
- Created labels: "outdoor", "personal", "cameras"
- Applied labels systematically across relevant devices

**Critical Finding:** Labels don't enable dynamic dashboard filtering in HA (label functionality limited to organizational tagging in device/entity lists). Does not solve dashboard complexity at scale as initially hoped.

---

## Wyze Integration (Dual-Path)

**HACS Custom Integration:** Doorbell + motion sensor integration for automation triggers (weekend automation functional)

**Docker Addon:** Camera feed access

**Motion Detection Toggle Issue:**
- Observed behavior: HA sets toggle state → Wyze app shows different state → HA reverts
- Root cause uncertain: Cloud API rate limiting, polling conflict, or feature may be state indicator rather than control switch
- Current solution: IFTTT handles intended automation functionality
- Migration priority: Need HA-native solution before canceling IFTTT subscription (planned transition from IFTTT → full HA consolidation)

---

## System Maintenance Events

**Synology NAS Drive 2 Failure (ongoing monitoring):**
- Initial incident: Beeping, critical errors after improper shutdown, initially suspected loose enclosure bay
- Current understanding: Drive 2 crashes periodically → volume becomes read-only (including Drive 1) → blocks backups → restart required → eventually crashes again
- Status: Monitoring backup status every few days, cloud backup current, new drive purchase planned (not urgent)
- **Critical validation:** Drive crashes don't destabilize HA with Samba Backup addon (confirms S6 architecture decision to avoid network storage mount dependencies)

**Proxmox Storage Investigation:**
- Red 98% warning displayed in Proxmox storage overview
- Investigation: LVM thin provisioning shows allocated vs used space (VMs reserve space on creation, only consume when data written)
- Actual usage: ~1-5% across all storage partitions (root, local, local-lvm)
- Resolution: Known Proxmox UI display behavior, system healthy, no action needed

**HA Memory Allocation Increase:**
- Initial: 88.18% (3.53GB/4GB) - functioning properly but less headroom for spikes
- Resize: Shutdown VM → Proxmox Hardware tab → 4GB → 6GB → Start
- Post-resize: 70.27% (4.22GB/6GB)
- HA dynamically utilized additional memory for better performance (caching, database queries, integration states)
- Enables: Better handling of simultaneous automations, improved dashboard loading, buffer for integration data spikes

---

## Critical Issues Discovered

**SmartThings Bridge Reliability:**

*Automation:* "Outside Lights On"
- Trigger: Sunset-based
- Target devices: 5 lights (including Tenant Side Door Light)
- Runtime: < 1 second

*Observed Pattern:*
- 5 device actions attempted
- 4 show "turned on" in trace
- Tenant Side Door Light CONSISTENTLY MISSING from success list (complete command failure, not false positive)

*Device Validation Testing:*
- Manual control via HA: ✅ Works
- Control via SmartThings app: ✅ Works  
- Automation via HA: ✗ Fails consistently

*Pattern Analysis:*
- Nightly automation: Tenant device failure pattern consistent
- Conclusion: Timing-dependent, not device hardware issue

*Root Cause:* SmartThings cloud bridge drops commands in rapid sequence - HA sends 5 → ST processes 4, drops 1 consistently, no error reporting visible in HA

*Current Workaround:* Automation disabled in HA, running from SmartThings app directly (temporary, defeats HA consolidation purpose)

*Network Context:* SmartThings hub connected via ethernet

**SmartThings Entity Exposure Limitations:**

*Discovery:* SmartThings integration only passes basic on/off controls to HA, does not expose device metadata or advanced entity attributes

*Example - Light Switches (Known Dimmers):*
- Can manually dim in HA UI (slider control works)
- NO dimmer level entity exposed for programmatic control
- Cannot check brightness level in automations
- Cannot set specific brightness values in automations
- Only on/off state available for automation logic

*Missing Across All SmartThings Devices:*
- Manufacturer information
- Firmware version
- Device model details
- Power monitoring (even when device supports it)
- Advanced configuration entities

*Impact:* Cannot create brightness-based automations, cannot monitor device health/firmware, cannot implement advanced control logic using full device capabilities, building automations blind to actual device features

*Strategic Implication:* Significantly impacts Session 9 automation expansion planning - current ST bridge approach limits what's possible vs direct device integration

---

## Technical Troubleshooting Documentation

**Multiple HA Windows Editing Conflict:**

*Problem:* Separate browser windows editing HA simultaneously causes save conflicts

*Root Cause:* HA doesn't handle concurrent editing well across separate windows

*Recommended Workflow:*
- Single window, multiple tabs (not separate windows)
- Tab 1: Settings → Devices & Services → Entities (entity ID reference)
- Tab 2: Dashboard editor
- Switch between tabs to copy IDs and paste in dashboard
- Avoid simultaneous dashboard edit sessions

**Entity Picker Usability at Scale:**

*Problem:* HA entity picker doesn't scale well with 148+ entities - search returns long lists on partial match, scrolling time-consuming, visual picker cumbersome for dashboard creation

*Workflow Solutions:*
- Tab 1: Settings → Devices & Services → Entities → sort/filter for exact entity IDs
- Tab 2: Dashboard editor → copy IDs from Tab 1 and paste
- Browser search: Ctrl+F when entity picker opens (more precise than picker's built-in search)
- Dashboard YAML mode: Skip visual editor, paste entity IDs directly, bypasses picker
- Temporary "scratch" dashboard: Experimentation space → copy successful cards to production

*User Mitigation:* Systematic entity renaming as devices added (already completed) helps search functionality

*YAML Alternative:* Presented as alternative to visual editor frustration, user preference to continue visual for now (annoying but manageable), YAML deferred until specific need arises

---

## Resolved Items

**Samba Backup Router Reboot Test:**
- Previous SMB mount attempts crashed HA during network interruptions
- Current Samba Backup addon uses push model (HA → NAS) vs mount model (NAS mounted in HA)
- Result: ✅ Backup survived router automatic reboot, HA remained stable, no crashes since implementation
- Validates S6 backup architecture decision

**Google Calendar Integration:**
- Weekend event automation tested and confirmed functional
- Status: ✅ Operational

---

## Outstanding Items (Session 8)

**Honeywell T5 OAuth Approval:**
- Submitted 3+ weeks ago, still pending
- Priority: Non-critical, resolution deferred

**Status Dashboard:**
- Deferred pending ZWave migration strategy results
- Decision: Wait to understand final device architecture before building comprehensive status view

**Health/Personal Tracking Dashboard:**
- Explicitly deferred
- Priority: Lower than household automation

---

## Resources

- [HACS Documentation](https://hacs.xyz/docs/use/)
- [SmartHomeJunkie Google Calendar Tutorial](https://smarthomejunkie.net/how-to-use-calendar-events-in-home-assistant-tutorial/) (adapted for Google OAuth changes)
- [HA Fitbit Integration](https://home-assistant.io/integrations/fitbit/)

---

## Network Architecture Current State

| Component | IP | Status | Notes |
|-----------|-----|--------|-------|
| PVE Host | 192.168.50.X | Stable | Sleep disabled, 27+ day uptime |
| HA VM | 192.168.50.X | Running | 2 CPU, 6GB RAM (↑ from 4GB), 32GB disk, DHCP reservation, HAOS 16.2 |
| Synology NAS | 192.168.50.X | Monitoring | DS218j, NFS + Samba Backup, RAID 0, Drive 2 periodic crashes (non-HA-impacting), cloud sync active |
| Router | - | Active | ASUS ZenWiFi, IoT isolation disabled, Monday 3AM auto-reboot |

**Integration Paths:**
- ZWave: SmartThings → HA (cloud bridge)
- Zigbee: SmartThings → HA (cloud bridge)
- WiFi devices: Direct HA (local)
- Ecobee (3): Direct HomeKit (local)
- August: Direct HomeKit (local)
- Mysa: Direct HomeKit (local)
- LG ThinQ: Direct cloud API
- Fitbit: Direct cloud API
- Wyze: HACS custom integration + Docker addon
- Google Calendar: Direct cloud API (OAuth)
- Honeywell T5: Pending OAuth approval
- Future ZWave: Direct HA via USB dongle (hardware validated S5-6, migration pending test)

---

## Device Inventory

**Total Entities:** 148+

**SmartThings Bridge:**
- ~35 ZWave light switches/dimmers
- ~10 Zigbee smart plugs

**Direct Integrations:**
- HomeKit: 3 Ecobee thermostats, 1 August lock, 1 Mysa thermostat
- WiFi: Multiple Kasa devices (TP-Link)
- Cloud: LG washer/dryer, Fitbit
- Wyze: Doorbell, motion sensors, cameras (HACS + Docker)
- Reolink: NVR + individual cameras
- Google Cast: Media devices
- Google Calendar: Event-based automation triggers

**Entity Naming Convention:**
- Systematic rename completed for all devices as added
- Device type suffixes applied ("_light", "_plug", "_switch")
- Areas removed from non-location devices (personal tracking separated from location-based automation)

---

## Key Learnings

**Technical:**
- Physical device reset clears pairing state when software methods fail (Mysa hold-arrow-button solution after 7 failed software attempts)
- HA caching can prevent entity updates from persisting, multiple browser windows may contribute, systematic refresh between changes required
- UI automation entity reference updates automatic, YAML requires manual ID updates
- Label functionality in HA limited to organizational tagging, doesn't enable dynamic dashboard filtering
- SmartThings cloud bridge drops commands in rapid sequence (timing-dependent failures)
- SmartThings integration entity exposure severely limited vs direct integration (basic on/off only, missing advanced controls and metadata)
- HA entity picker poor scalability at 148+ entities (workflow adaptations required)

**Project Management:**
- Hours-long tasks can hide behind "simple" descriptions (entity management caching reality)
- Physical device troubleshooting sometimes faster than complex software debugging (Mysa 7 attempts vs 1 reset)
- Dashboard architecture strategy more important than individual dashboard implementation (functional > categorical validated early prevents wasted build time)
- Hardware failures happen but proper architecture isolation prevents cascades (NAS crashes don't destabilize HA validates S6 decision)

---

## Next Session (S9 Planned)

**Phase 1: Monitoring & Notifications Setup**
- System health monitoring (CPU, memory, storage thresholds)
- Device failure/offline detection
- Backup status verification
- Automation failure tracking
- Mobile app notifications
- Critical email alerts (HA crashes, backup failures, resource exhaustion)

**Phase 2: ZWave Single-Device Test (if Phase 1 stable)**
- Migrate Tenant Side Door Light (SmartThings → Direct ZWave via USB dongle)
- Evaluate entity exposure improvements vs ST bridge
- Test automation reliability with direct ZWave control
- Validate monitoring system catches issues during migration

**Decision Point:**
- ZWave shows clear value → S10 = full migration planning
- ZWave marginal improvement → S10 = alternative approaches (dashboard expansion, automation optimization)
