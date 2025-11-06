# SESSION 11: ZWAVE MIGRATION & AUTOMATION ARCHITECTURE (work across ~1 week)

**Status:** ✅ Complete
**Achievement:** 29 ZWave devices migrated to direct HA control, automation architecture established to survive device migrations, system operational after mesh stabilization troubleshooting

---

## Session Scope

S11 encompassed both automation architecture work and physical ZWave device migration. Core challenge: migrate devices from SmartThings to HA without breaking existing Google Home voice automations and time-triggered routines. Solution required designing label-based Script architecture before migration, then migrating 29 devices (25 switches, 4 scene controllers) from ST cloud bridge to direct HA ZWave JS control. Post-migration work included mesh routing troubleshooting, reliability improvements, and mobile app remote access configuration.

---

## Automation Architecture: Migration-Survival Strategy

### The Core Problem

Google Home automation architecture creates migration fragility:
- When device unpaired from SmartThings, Google Home auto-deletes all references
- No trace of which automations used that device
- Requires rebuilding automations from memory after migration
- With 16+ Google routines and SmartThings automations, manual recreation risk is high

Need architecture that survives migration: device underneath changes, automation layer remains intact.

### Solution: Scripts + Labels in HA

**Architecture:**
- Google Home automations call HA Scripts (via Nabu Casa Google Assistant integration from S10)
- Scripts target devices by labels, not individual entity IDs
- Labels applied to ST-bridged devices before migration
- When ST device removed: HA Script shows unavailable entity (visible, correctable)
- When HA device added with same label: Script automatically includes new device
- Google Home automations unchanged throughout migration

**Benefit:** Decouples automation logic from device pairing. Migration becomes: unpair device, repair device, apply same label. Automations continue working.

### Label + Template Architecture Decision

**Challenge encountered:** Simple label targeting insufficient for complex logic
- Need: outdoor lights that auto-enable at night (area + label AND logic)
- Need: all cameras except specific rooms (label + area exclusion)
- HA UI: single label selection only, no AND/OR/exclusion logic

**Alternatives evaluated:**
- Entity-based labels: works in UI, but tedious at scale (150+ entities vs 29 devices)
- Direct entity selection: no scalability, hard-coded device lists in every Script
- Stay with simple labels: limits automation capability, can't express complex logic

**Decision:** Device-based labels with Jinja2 templates for filtering

**Rationale:**
- Devices faster to label than entities (one label per device vs multiple per device)
- Templates enable complex logic: area + label combinations, exclusions, multiple label AND
- Copy/paste pattern once established
- Future-proof for Zigbee, Matter devices using same label structure

**Validation approach:** Created proof-of-concept with 4 test devices, verified template extracted entities correctly, tested via Google voice automation, confirmed logging showed individual device names for troubleshooting. Template executed reliably. Committed to architecture.

**Tradeoff accepted:** Requires YAML editing for template-based actions. UI doesn't support complex filtering. Learning curve for template syntax mitigated by reusable pattern.

### Pre-Migration Automation Work

Completed before physical migration started:
- Migrated 16 Google Home voice routines to HA Scripts
- Migrated SmartThings automations to HA Scripts/automations
- Moved time-triggered automations from ST to HA
- Applied labels to ST-bridged ZWave entities
- Validated all automations functional with ST devices before touching hardware

Result: Migration became device swap exercise. Automation layer remained stable.

---

## Physical ZWave Migration

### Migration Strategy

**Approach:** Priority-based migration
- Problem devices first (identified from HA automation logs showing ST bridge failures)
- High-use devices next
- Remaining devices as time permitted

**Rejected:** Geographic every-other-room pattern. Impractical given device distribution, added planning overhead without clear benefit.

**Risk accepted:** Potential mesh routing gaps during transition. Believed adding devices would strengthen mesh, not degrade it.

### Devices Migrated

**Total: 29 devices**
- 25 switches (mix of S2 and older non-S2 generation)
- 4 scene controllers (battery-powered, 2-button, added for physical automation triggers in areas without convenient switch access)

Non-S2 devices work without issues in mixed-generation mesh.

**QR code documentation:** Photographed S2 device QR codes during initial pairing. Avoids needing to unscrew faceplates if device requires re-adding due to dead node issues encountered post-migration.

### Pairing Challenges

**Most unexpected difficulty:** Software pairing unpredictability, not physical work.

Expected unscrewing faceplates to dominate timeline. Reality: drill made physical work quick. Pairing software (exclusion and inclusion) was bottleneck.

**SmartThings exclusion inconsistency:**
- Some devices excluded immediately
- Others required multiple retry attempts
- Some required physical switch reset
- Some required waiting and revisiting later
- No diagnostic feedback on why exclusion fails or how to resolve
- Time varied: some quick, some took extended troubleshooting

**HA ZWave JS inclusion issues - more painful than ST exclusion:**
- Inconsistent pairing success requiring guessing different approaches
- One device required breaker cycling to successfully include
- Duplicate device creation bug: unnamed device functional, named device non-functional until unnamed removed
- Ghost node (1 device): wouldn't remove from ZWave JS, froze physical switch, required breaker reset + physical switch reset + manual node removal to resolve
- Time varied significantly with no predictable pattern

### Post-Migration Mesh Stability

**Scope management decision:** Continued full migration despite stability issues.

Expected one-time pain getting devices onto system, then stability. Believed adding more devices would improve mesh coverage. Standard setup used by many, should work once established. USB extender ordered to test RF interference hypothesis, will evaluate in S12.

**Initial problems (Days 1-3 after migration):**
- Devices showing as "dead" despite successful pairing
- Automations failed or skipped random devices
- Some devices slow to respond
- Some devices reported state change in HA but didn't physically execute

**Mesh routing issues identified:**
- Mesh map showed connections from opposite sides of house
- Devices not listed as neighbors attempting direct communication
- Geographic logic didn't match routing decisions

**Troubleshooting attempts without success:**
- Individual node mesh repair
- Full mesh rebuild
- HA restart
- Waiting for auto-optimization

**Manual routing intervention (Days 3-5):**
- Identified problem devices via mesh visualization and observed behavior
- Manually configured routes through geographically closer devices
- System achieved reliable performance after ~5 days total

**Current state:** Manual routes remain in place. Will reassess after USB extender evaluation (testing if RF interference from USB 3.0 port near laptop is contributing factor). System currently stable.

### ZWave Controller Evaluation

Research into alternative controllers during troubleshooting:
- Current Zooz ZST10 700 series: no external antenna, USB-based requiring proximity to laptop running Proxmox/HA, no 800-series support (not current problem but limits future compatibility)
- Firmware 7.18.3 supposed to fix mesh issues but stability concerns persist
- Questioning whether issues are USB stick hardware vs HA ZWave JS integration
- Blogs document similar issues with USB-based sticks at scale

**Decision:** Keep current stick, try USB extender mitigation first. Reevaluate if extender doesn't improve stability. Would choose different controller (non-USB-based or with external antenna) if starting over, but not replacing functional equipment without data.

---

## Reliability Improvements

### Intermittent ZWave JS Driver Failures

**Issue discovered:** Automation executes, most devices work, random device fails with driver crash.

Error: `TypeError: resp.toSupervisionResult is not a function` during command execution. Device available, command sent, driver crashes. Retry immediately succeeds.

**Impact:** Time-triggered automations unreliable when running unattended (sunrise/sunset, away mode while not home).

### Retry Logic Implementation

**Decision:** Implement immediately rather than defer to S12 monitoring.

Rationale: Time-based automations run unattended. Cannot wait to quantify problem when failures impact daily operation. Need reliability now.

**Approach:** Wrap automation sequences in retry loop (count: 2, delay: 2 seconds between attempts). Run automation twice. Driver crash typically doesn't repeat on second attempt.

Applied to all time-triggered and device-triggered automations. Not applied to Google voice Scripts (present to manually retry if needed).

Result: Automations execute twice. Idempotent operations (turn off already-off light) have no negative effect. Adds 2-second overhead. Ensures reliability for unattended operation.

### Ongoing Device Unavailability

**Pattern observed:** Devices intermittently show as "dead" in HA ZWave JS over migration week. Frequency unclear (insufficient data). Affected devices appear random.

**Recovery methods attempted:**
- Wait for automatic recovery (sometimes works)
- Ping device via ZWave JS (often successful)
- Re-interview device (unclear if effective or coincidental with auto-recovery)
- Delete and re-add device (requires physical reset, most invasive - QR photos prevent needing to unscrew faceplate)

**Root cause unclear - multiple hypotheses:**
- Mesh routing table synchronization issues (ping often resolves)
- RF interference from USB 3.0 port (extender ordered to test)
- ZWave stick hardware limitations at 29+ device scale
- Manual routing conflicts with auto-optimization
- Electrical noise on power lines
- Device hardware quality variation

Likely multiple contributing factors. Need monitoring data (S12) to identify patterns before committing to solutions.

---

## Nabu Casa Remote Access

### Mobile App Configuration

Mobile app worked on home WiFi but failed on cellular with connection error.

Root cause: App configured with only internal URL. Missing external URL and WiFi network identifier for automatic switching.

Resolution: Configured single server entry with both internal URL (local network) and external URL (Nabu Casa), plus WiFi SSID to trigger switching. App now routes correctly based on network connection.

UI confusion: Internal URL field greyed out until WiFi network set. Not obvious that WiFi name is prerequisite.

### Google Home Manual Control Issue

Manual device control in Google Home app requires double-tap for ZWave devices. First tap executes physically but UI reverts. Second tap updates UI correctly. Voice commands work without issue.

Root cause: ZWave mesh routing delay causes Google timeout on state confirmation. Google assumes command failed, reverts UI.

Persistent over week-plus observation. Not cache issue, won't self-resolve. Accepted as limitation. Voice commands work reliably (primary use case). HA mobile app control via dashboards (S13) will provide alternative.

---

## Deferred Scope

**Tenant ZWave devices (9 switches):** Located opposite side of house from USB controller with controller positioned between two device sets. Currently functional via ST bridge. Older non-S2 generation. Migrate when convenient.

**Zigbee devices (9 devices):** Remaining on ST bridge. Plan to test manual Script delays for timing mitigation before deciding Zigbee coordinator purchase.

**Device requiring electrician (1 switch):** Needs rewiring. Deferred.

**Available scene controllers (2 devices):** No immediate deployment use case identified.

---

## Key Learnings

**Scripts + labels architecture survives device migrations:** Google Home automations unchanged throughout 29-device migration. HA Scripts showed unavailable entities when ST devices removed, automatically included new devices when HA entities labeled. Decoupling automation logic from device pairing eliminated migration fragility.

**ZWave pairing unpredictability dominated timeline:** Expected physical work (unscrewing) to be bottleneck. Software pairing (exclusion/inclusion) was actual constraint. No diagnostic feedback on failures. HA inclusion more painful than ST exclusion, requiring guessing approaches. Time per device varied significantly with no predictable pattern.

**Mesh stabilization required after migration:** Expected one-time pairing pain, then stability. Reality: 5-day troubleshooting period with manual routing intervention. Auto-optimization didn't resolve routing issues. Manual routes based on mesh visualization and observed behavior (slow response, non-execution despite HA confirmation).

**Template architecture enabled complex automation logic:** Device-based labels with Jinja2 templates provided area + label filtering, exclusions, multiple label AND logic not possible in UI. Copy/paste pattern acceptable for ~20 Scripts. Validated with proof-of-concept before committing.

**Retry logic cheaper than root cause analysis:** 2-second retry overhead vs extended troubleshooting of intermittent ZWave JS driver crash. Pragmatic solution for home automation reliability. Driver crash typically doesn't repeat on second attempt.

**Priority-based migration more practical than geographic patterns:** Target known problem areas first. Iterate based on observed failures. Less planning overhead. Can pause at any point.

**USB-based ZWave stick limitations at scale:** No external antenna, proximity to laptop required, USB 3.0 RF interference concern, no 800-series support for future devices. Would choose different controller if starting over. Keeping current stick, testing USB extender mitigation. Questioning if issues are stick hardware vs HA ZWave JS integration. Need data before replacing functional equipment.

**QR code photography during initial pairing:** Forward-thinking preparation. Photographing S2 device QR codes during installation avoids needing to unscrew faceplates when dead node requires device re-addition.

---

## Outstanding Items

**S12 - Monitoring & Notifications:**
- Comprehensive HA notification strategy
- Integration monitoring
- Device offline alerts (ZWave device availability tracking, identify failure patterns)
- Automation failure detection (Script executed but device didn't respond)
- System health monitoring
- Correlation analysis (manually-routed vs auto-routed devices, time of day, automation activity)
- USB extender impact evaluation

**S13 - Dashboard Development:**
- Provides HA mobile app control alternative to Google Home manual control
- Status dashboard
- Functional dashboards

---

**Next session:** S12 - Comprehensive monitoring and notification strategy
