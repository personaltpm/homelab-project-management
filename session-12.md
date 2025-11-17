# SESSION 12: COMPREHENSIVE MONITORING INFRASTRUCTURE (incremental work across ~2 weeks)

**Status:** ✅ Complete
**Achievement:** Operational monitoring system covering device health (battery/offline), system resources (CPU/memory/disk/load), automation errors, and backup failures with intelligent notification strategy via Telegram

---

## SESSION SCOPE

S12 implemented comprehensive monitoring infrastructure to detect issues early across all system layers. Core challenge: balance real-time alerting for critical failures against notification spam from normal operational patterns. Solution required understanding which metrics benefit from immediate alerts vs daily summaries, establishing notification hierarchy, and creating custom automations when blueprint limitations blocked complex exclusion logic.

Monitoring coverage implemented: 9 automations operational across device health monitoring (battery levels, offline detection for battery and hardwired devices), system resource monitoring (CPU, memory, disk, system load), automation error detection, and backup failure alerts. Notification delivery via Telegram with appropriate real-time vs daily scheduling based on metric timescale and actionability.

---

## DECISION: BLUEPRINTS VS CUSTOM AUTOMATIONS

**Problem encountered:** Started with community blueprints for low battery and offline device monitoring. Hit multiple blocking limitations during implementation.

**Blueprint limitations discovered:**
1. Inflexible exclusions: Couldn't exclude devices without battery entities (Wyze cameras, Vesync humidifier incorrectly reporting as battery devices)
2. No logic modification: Template logic embedded in blueprint, couldn't adjust without breaking blueprint updates
3. UI selector constraints: Only showed battery-class entities in exclusion picker, couldn't manually add entity_ids

**Alternatives considered:**
- Continue with blueprints, accept limitations: Require workarounds for exclusions
- Fork blueprints: Lose automatic updates, still constrained by blueprint structure
- Custom automations: Full control, more verbose YAML, manual maintenance

**Decision:** Migrate to custom automations for all monitoring

**Rationale:**
- Need flexible exclusion logic blueprints couldn't provide
- Monitoring logic stable (infrequent changes expected), losing blueprint updates acceptable
- Documentation value: logic explicit in project, not hidden in blueprint abstraction

**Tradeoffs accepted:**
- More verbose YAML (acceptable for clarity)
- Manual maintenance (acceptable for stable monitoring logic)
- No automatic blueprint improvements (acceptable, monitoring needs specific to setup)

---

## DECISION: DAILY VS REAL-TIME MONITORING STRATEGY

**Core question:** Which metrics need immediate alerts vs daily summaries?

**Analysis approach:** Match monitoring frequency to metric timescale and actionability.

**Real-time monitoring (immediate alerts):**
- **Disk space >85%:** HA crashes at 100%, can't fix if offline - need early warning before critical
- **Device offline (hardwired):** Locks, thermostats need immediate attention, especially when away from home
- **Device back online:** Confirms recovery without manual checking
- **Automation errors:** Script failures need immediate visibility when automations run unattended
- **Backup failures:** Know immediately when nightly backup doesn't complete

**Daily monitoring (9am/10pm summaries):**
- **Battery levels:** Drain slowly (days/weeks timescale), daily check sufficient for replacement planning
- **Battery device offline:** Overlap with low battery detection, not actionable more frequently
- **System health (CPU/memory/disk/load):** Temporary spikes normal during backups/automations, care about sustained patterns not momentary activity
- **Hardwired devices offline (summary):** Daily list shows persistent problems, complements real-time critical alerts

**Rationale for split strategy:**
- Real-time: Prevents outages, actionable immediately
- Daily: Planning horizon, trend detection, reduces notification fatigue
- Battery devices specifically: Real-time would wake sleeping devices (wastes battery monitoring is trying to preserve)

**Decision:** Implemented both real-time and daily automations based on metric characteristics, not blanket approach.

**Notification hierarchy implementation:**
- Created two Telegram groups: standard notifications (muted) and Critical Alerts (sound enabled)
- Standard: Daily summaries, low battery alerts, offline device lists
- Critical: Real-time failures (disk >85%, automation errors, backup failures, critical device offline)

**Rationale:** Notification fatigue from frequent alerts reduces effectiveness. Critical alerts with sound for actionable immediate issues, muted for informational/planning items. Allows checking standard notifications when convenient without interruption, ensures critical issues get immediate attention.

---

## DECISION: SYSTEM MONITOR CONFIGURATION

**Integration choice:** Built-in System Monitor vs Glances

**Glances evaluation:**
- Provides detailed per-process, per-container breakdown ("why" not just "what")
- Shows which add-on consuming resources when alerts trigger
- Network I/O per container
- **Requires:** Disabling Protection Mode (security concern)

**System Monitor characteristics:**
- Provides metrics only ("what" - disk 90% full, CPU 80%)
- No root cause visibility (which process, which container)
- No Protection Mode compromise
- Sufficient for alerting

**Decision:** System Monitor for monitoring, Glances enabled temporarily when troubleshooting

**Rationale:**
- Monitoring goal: Know when problems occur
- Root cause analysis: Secondary step after alert triggers
- Security: Protection Mode matters for public documentation project
- Pragmatic: Enable Glances on-demand rather than permanent security tradeoff

**Sensors enabled (rationale):**
- Core metrics: `processor_use`, `memory_usage`, `disk_usage` for alerting thresholds
- Sustained stress: `load_15_min` (15-min average filters temporary spikes from backups/automations)
- Reboot tracking: `last_boot` for unexpected restart detection
- Troubleshooting history: `network_throughput_in/out_hassio`, `disk_usage_media`, `disk_usage_config`

**Key insight:** Historical troubleshooting sensors enabled before problems occur. When alert triggers (disk 90% full), can review past days to see what changed. Current-only data doesn't show root cause.

---

## MONITORING COVERAGE IMPLEMENTED

**Device monitoring (4 automations):**
- Daily low battery check (threshold 20%)
- Daily offline battery device check
- Real-time hardwired device offline (5-min delay filters transient network issues)
- Real-time device back online confirmation

**System monitoring (2 automations):**
- Daily health check: CPU >80%, memory >80%, disk >75%, load >4
- Critical disk alert (real-time >85%)

**Infrastructure monitoring (3 automations):**
- Real-time automation/script error detection (requires `system_log: fire_event: true` in configuration.yaml)
- Real-time backup failure alerts
- Daily backup overdue check (2+ days without successful backup)

**Notification formatting:**
- Plain text via Telegram (HTML caused syntax complications)
- Device names not entity names for cleaner output
- Alphabetized lists for scanning
- Mode: single to prevent duplicate notifications from multi-entity devices

---

## ZWAVE MESH STABILITY RESOLUTION

**Outstanding issue from S11:** Intermittent device unavailability, multiple hypotheses (mesh routing, RF interference, stick hardware, electrical noise)

**USB extender testing (S12):**
- Hypothesis: USB 3.0 RF interference from laptop port affecting ZWave stick operation
- Solution: USB extender providing physical distance from USB 3.0 ports
- Result: Mesh routing stabilized, intermittent unavailability significantly reduced

**Monitoring approach:** Continue tracking device availability patterns via S12 monitoring infrastructure to validate sustained improvement and identify any remaining edge cases.

---

## KEY LEARNINGS

**Blueprint limitations emerge with complexity:** Simple use cases work well. Complex logic (custom exclusions, conditional notifications) better served by custom automations despite more verbose YAML. Control and maintainability outweigh brevity.

**Real-time vs daily strategy matches metric timescale:** Fast-changing critical metrics (disk space, device offline) benefit from immediate alerts. Slow-changing trends (battery levels, system load averages) better served by daily summaries. Reduces notification fatigue while maintaining coverage.

**Historical sensor data enables root cause analysis:** Enable troubleshooting sensors (disk usage breakdown, network throughput) before problems occur. When alerts trigger, historical data shows what changed. Current-only metrics don't reveal root cause.

**"What" vs "Why" monitoring security tradeoff:** System Monitor provides metrics sufficient for alerting without Protection Mode compromise. Glances provides per-process diagnostics requiring Protection Mode disabled. Enable Glances temporarily when troubleshooting rather than permanent security tradeoff.

**Mode: single prevents duplicate notifications:** Essential for device-level alerts when multiple entities per device exist (cameras with power/motion/notification switches). First entity triggers, subsequent ignored until completion.

**USB extender resolved RF interference:** USB 3.0 RF interference from laptop port to ZWave stick caused intermittent device unavailability. USB extender (physical distance from USB 3.0 ports) resolved mesh stability issues that manual routing couldn't fully address.

---

## OUTSTANDING ITEMS

**Proxmox monitoring (deferred to future session):** Monitor Proxmox host and VM resources via Proxmox VE integration. Tracks host CPU/memory/disk, VM status. Separate from HA system monitoring (monitors hypervisor layer).

**Synology NAS monitoring (accepted as sufficient):** Built-in NAS notifications cover disk health, service status. Centralized monitoring in HA possible via Synology integration but lower priority. Current NAS alerts adequate.

---

**Next session:** S13 - Comprehensive system monitoring, area, user, and action based dashboards
