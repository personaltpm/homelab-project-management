# SESSION 10: External Access & Voice Automation Foundation (work-across-multiple-days)
**Status:** ✅ Complete
**Achievement:** Nabu Casa operational with external access validated, Google Assistant integrated, voice automation architecture established via Scripts, NAS hardware replaced with 3TB data restored and backup cycles re-established

## NAS Hardware Replacement & Data Restore

**Challenge:** Drive 2 weekly crashes in RAID 0 configuration, 6TB replacement drive purchased (S9), need to migrate data without local storage capacity for intermediate copy.

**Hardware swap sequence:**
1. Shut down NAS, removed both drives (labeled "RAID0-backup-Drive1/2-[date]" for disaster recovery)
2. Inserted 6TB drive, powered on
3. Volume configuration: selected **SHR** (Synology Hybrid RAID) over Basic
   - Min 1 drive, allows future redundancy addition without data migration
   - When 2nd drive added: automatically enables fault tolerance
   - Chose SHR for future flexibility vs Basic for separate volumes

**iDrive restore strategy:**
- **Test restore:** 3.52GB in 4:33 minutes = ~46GB/hour
- **Estimated full restore:** 3TB / 46GB/hour = ~67 hours (~2.8 days)
- **Actual timeline:** ~4 days (incremental validation approach)
- **Method:** Restored folders incrementally (HA, Proxmox, photos, mom's backups) for visual confirmation
- **Validation:** Ran full restore twice - first for initial copy, second verified existing files not re-copied (integrity confirmation)
- **Result:** All 4 folders restored successfully, NAS stable on new 6TB drive

**Post-restore reconfiguration:**

**Synology Telegram notifications:**
- Recreated shared folders, user permissions (stricter password requirements in new DSM)
- Control Panel → Notification → Push Service → Webhooks
- **Issue:** Webhook URL had "-api" typo in bot token (should be `bot<TOKEN>` not `botAPI-<TOKEN>`)
- **Resolution:** Removed "-api" suffix, test message successful
- Re-enabled all notification types (validated with real Drive 2 crash trigger in S9)

**Proxmox NFS backup:**
- **Issue:** NFS service not running after NAS restart (service enabled but not started)
- **Diagnostic:** `showmount -e 192.168.50.190` returned "Connection refused"
- **Resolution:** Restarted NAS, NFS service initialized properly
- Recreated NFS storage in Proxmox (Datacenter → Storage → Add NFS)
  - ID: synology-bkup (note: originally "synology-backup" in S6, renamed during recreation)
  - Export: /volume1/proxmox-bkups
  - Content: VZDump backup file
- **Validation:** Manually triggered backup via Datacenter → Backup → Run now, successful write to NAS

**HA Samba Backup addon:**
- Updated credentials in addon config (NAS enforced stricter password policy)
- Share: ha-bkups, User: homeassistant
- **Validation:** Backup successful, architecture from S6 (addon pushes TO NAS) remained stable throughout NAS crashes

**Backup status:** Proxmox + HA automated backups ran successfully throughout 4-day restore period, no interruptions to backup cycles.

**Lesson:** iDrive restore process reliable but time-intensive, incremental validation approach (restore twice, folder-by-folder) provided confidence in data integrity vs single bulk restore.

## External Access & Platform Decision

**Nabu Casa vs DIY evaluation:**

**Requirements:**
- External access to HA (remote control, monitoring)
- Google Assistant integration (voice control, existing smart home ecosystem)
- Security without self-managed certificate/DNS complexity
- Time efficiency (avoid maintenance overhead)

**Decision: Nabu Casa ($65/year, 31-day trial)**

**Rationale:**
- IFTTT cancellation ($3.99/month = $47.88/year) → Nabu Casa $65/year = ~$17 additional for superior functionality
- Security concerns with DIY (DuckDNS + Let's Encrypt + port forwarding) eliminated
- Maintenance overhead eliminated (no certificate renewal, no DNS management, no router config)
- Time saved = worth cost differential
- 31-day trial = risk-free validation period
- Funds HA development (aligns with "build public" project value)

**Alternatives considered:**
- DuckDNS + Let's Encrypt: free but complex setup, ongoing maintenance, security risk if misconfigured
- Cloudflare tunnels: additional service dependency, learning curve
- VPN-only access: no Google Assistant integration, less convenient

**Tradeoffs accepted:**
- Annual subscription cost vs free alternatives
- Dependency on Nabu Casa service (acceptable given HA local control remains functional)
- Cloud relay for external access (minimal latency observed)

**Setup:**
- Installed Nabu Casa integration via Settings → Home Assistant Cloud
- Configured account, enabled remote access
- **Validation:** External access confirmed via mobile network (non-WiFi), HA interface responsive

## Google Assistant Integration

**Configuration path:** Settings → Voice Assistants → Google Assistant (not Devices & Services integration)

**Initial setup:**
- Exposed 2 migrated ZWave lights (Tenant Side Porch, Side Light from S9)
- Google Home app → Add device → Works with Google → Home Assistant Cloud
- Devices appeared in Google Home, controllable via voice

**State sync issue discovered:**

**Symptom:**
- Manual control in Google Home app: light physically turns on, HA shows on, but Google Home UI reverts to "off"
- Second tap: works correctly, all systems sync
- Voice commands: work perfectly, no sync issues
- Control from HA → Google Home: syncs correctly

**Root cause:** Google Home state reporting race condition
- Google sends command → HA executes → light responds
- Google expects immediate state confirmation
- Z-Wave mesh response + HA state report delayed
- Google assumes command failed, reverts UI
- Second tap works because HA state already "on" when Google queries

**Troubleshooting attempted:**
- Searched for "Request Sync" option (Google Assistant integration doesn't expose in newer HA architecture)
- Developer Tools → Actions tab (renamed from "Services") → `google_assistant.request_sync` not available (integration via Voice Assistants, not traditional integration)
- Considered unlink/relink in Google Home app (deferred as likely self-resolving)

**Resolution:** Accepted as temporary behavior, expected to stabilize within 24-48 hours as Google Home cache establishes device states. Voice commands work reliably (primary use case). Manual app control less critical, double-tap workaround acceptable short-term.

**Entity exposure consideration:** Only exposed light entities, not diagnostic entities (status, ping, round trip time, identify, firmware, scene) - those remain HA-only for management.

## Voice Automation Architecture

**Challenge:** Google Home has no search/filter for automations by device, no export functionality. When ZWave devices unpaired from ST → Google auto-removes all references (no trace of what was used where). Need maintainable architecture for S11 migration of 33+ devices.

**Decision: HA Scripts exposed to Google Assistant**

**Architecture:**
- Create Scripts in HA (sequential actions, conditions, parameters)
- Expose Scripts to Google Assistant (Settings → Voice Assistants → Manage Entities)
- Google Home automations trigger Scripts via voice commands
- All logic in HA, Google just handles voice interface

**Scripts vs Scenes vs Blueprints evaluation:**

**Scenes:**
- Static snapshot of device states at creation time
- No parameters, no logic, no conditions
- Must manually re-save if device states change
- Exposed to Google Assistant

**Scripts:**
- Sequential actions with logic (delays, conditions, if-then)
- Support parameters (brightness percentages, variables)
- Easy to edit actions without re-saving states
- Dimmer control: `brightness_pct: 0-100` or `brightness: 0-255`
- Exposed to Google Assistant

**Blueprints:**
- Reusable automation templates
- Overkill for voice-triggered routines
- More complex to create

**Why Scripts:**
- Maintainable: edit actions, not re-save device states
- Dimmer control with explicit percentages (vs Scenes capturing current state)
- Can add logic later (conditions, delays, choose blocks)
- S11 migration: Scripts show unavailable entities when ST devices removed, easy to find/replace with HA entities
- Google automations unchanged during S11 (just trigger same script name)

**POC implementation:**
- Created 3 test Scripts: bedtime, sleep, morning
- Exposed to Google Assistant
- Created Google Home voice automations triggering Scripts
- **Validation:** All 3 Scripts tested via voice commands, executed correctly, HA logs confirmed proper sequence

**Example Script structure:**
```yaml
sequence:
  - action: light.turn_on
    target:
      entity_id: light.bedroom_dimmer
    data:
      brightness_pct: 20
  - action: light.turn_off
    target:
      entity_id: light.living_room
```

**S11 migration strategy:**
- Create Scripts using ST-bridged ZWave entities (devices still on ST)
- When ZWave migrated to HA direct in S11: Scripts show unavailable entities
- Replace ST entity → HA entity in each Script (easy to find)
- Google automations unchanged (still trigger same Script)
- Minimizes rework: only touch each Script once (replacement step)

**Benefits over Google-native automations:**
- Centralized management (all logic in HA)
- Version control possible (YAML backups)
- Visibility into automation execution (HA logbook)
- Survives device migrations (HA shows unavailable, doesn't delete references like Google)
- Complex logic possible (multiple conditions, variables, templating)

**Lesson:** Voice interface (Google) + automation logic (HA) separation provides maintainability and flexibility. Google handles natural language processing, HA handles execution and state management.

## Outstanding Items

**Deferred to S11:**
- Full ZWave migration (33+ switches from ST bridge to HA direct)
- Complete voice automation Script creation (for automations using ZWave devices)
- Script entity replacement after ZWave migration

**Deferred to S12:**
- Comprehensive HA notification strategy (integration failures, device offline, automation failures, system health)

**Deferred to S13:**
- Dashboard expansion (status dashboard, additional functional dashboards)

**Monitoring:**
- Google Home manual control double-tap issue (expected to self-resolve within 24-48 hours)
- NAS Drive 1 (4TB, healthy) available for future redundancy (add after S11 completes, not before high-risk ZWave work)

## Current System State

**Network Architecture:**
- PVE: 192.168.50.X, stable, sleep-off
- HA VM: 192.168.50.X, 6GB RAM, haos-16.2, externally accessible via Nabu Casa
- Synology NAS: 192.168.50.X, 6TB single drive (SHR), iDrive daily backup running
- External access: Nabu Casa cloud relay, validated functional

**Integration Status:**
- Google Assistant: configured via Voice Assistants, 2 ZWave lights exposed, voice control operational
- Telegram: NAS + Proxmox + HA notifications active
- Backup cycles: Proxmox (Mon 2AM, NFS to NAS) + HA (daily 2AM, Samba Backup to NAS) + iDrive (daily offsite)

**Voice Automation:**
- 3 Scripts created (bedtime, sleep, morning)
- Exposed to Google Assistant
- Google Home automations triggering Scripts
- Architecture validated for S11 expansion

---

**Key Lessons:**

**Technical:**
- NFS service enabled ≠ service running after restart (explicit restart required post-hardware change)
- iDrive restore reliable but time-intensive (~4 days for 3TB), incremental validation provides integrity confidence
- Google Home state sync race condition common during initial pairing, typically self-resolves within 24-48 hours
- HA Scripts + Google Assistant integration provides maintainable voice automation architecture (logic in HA, voice in Google)
- SHR with single drive = no redundancy initially but allows future drive addition with automatic fault tolerance conversion
- Typos in webhook URLs (botAPI- vs bot) cause silent failures, test messages critical for validation

**Project Management:**
- Nabu Casa cost ($65/year) justified by time savings + security confidence vs DIY complexity
- Incremental restore (folder-by-folder, twice) vs bulk restore trades time for validation confidence
- POC with 3 Scripts before full migration validates architecture before committing to pattern
- Deferring voice automation Script creation (for ZWave devices) to S11 minimizes rework (create once with correct entities vs create, migrate, update)
- Hardware changes (NAS drive swap) during project require reconfiguration time (Telegram, NFS, Samba) - factor into timeline
- Google Home's device reference auto-deletion (when devices unpaired) vs HA's unavailable entity retention makes HA superior for tracking dependencies during migrations
