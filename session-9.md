# SESSION 9: Monitoring & Notifications + ZWave Migration Test

**Status:** ✅ Complete  
**Achievement:** Telegram notification infrastructure operational across all platforms (NAS, Proxmox, HA), 2 ZWave switches migrated with direct entity control and OTA firmware capability, automation reliability validated via sunrise test

---

## Notification Platform Selection

**Platform chosen:** Telegram

**Rationale:**
- Bidirectional capability (can send commands back to HA/Proxmox, no immediate use case but future-proofing)
- Multi-use platform (not single-purpose notification tool, has other applications beyond home automation)
- Easy protocol switching later if needed (notification types/logic already configured, just swap delivery method)

**End-to-end encryption evaluation:**
- Telegram bot messages encrypted in transit (TLS) but not end-to-end
- Telegram Inc. can technically read messages, as can government with legal demand
- Alternatives considered: Signal (no bot API), Matrix (complex self-hosted setup), Email (worse encryption than Telegram), purpose-built services (less functionality)

**Threat model assessment:**
- Primary risk: Notification content exposure to Telegram Inc. or via server breach
- Actual exposure: Home automation patterns, device states, system alerts
- Not sending: Credentials, codes, or sensitive personal data via notifications
- Not high-risk threat model (journalist, activist requiring E2E encryption)

**Decision:** Proceed with Telegram - convenience and bidirectional capability outweigh theoretical E2E encryption risk for home automation use case. TLS protects against network eavesdropping, `allowed_chat_ids` mitigates unauthorized command risk.

**Mitigations applied:**
- Vague notification content where appropriate ("Security event" vs detailed specifics)
- No credentials or sensitive data in notification messages
- Notifications as alerts, check HA UI for full details when needed

**Alternative if threat model changes:** Self-hosted Matrix/Element with E2E encryption (would require additional infrastructure and maintenance overhead)

---

## Synology NAS → Telegram Setup

**Prerequisites:** DSM 7.1+ required for webhook notification option (upgraded NAS version to access feature)

**Configuration:** Followed guide: https://medium.com/@nikolayburyak/synology-dsm-7-1-telegram-alerts-b9c27fcbf8de

**Notification types:** Selected all types from list (easier to remove if spammed than add individually)

**Testing:**
- Test message: ✅ Sent successfully
- Real-world validation: ✅ Drive 2 crash triggered notification (see NAS Hardware Issue below)

**Status:** Production notifications unconfirmed until organic events occur (no errors/backups since configuration), but mechanism validated via test and actual failure

---

## Proxmox → Telegram Setup

**Initial approach:** URL query string with manual encoding
```
&text={{ url-encode "⚠️PBS Notification⚠️" }}%0A%0ATitle:+{{ url-encode title }}
```
- Messy, hard to read
- Manual URL encoding required
- Special characters problematic

**Error encountered:** `invalid uri character` when backup failed

**Solution:** JSON POST body with proper header

**Configuration:**
```
Header: Content-Type: application/json
Body:
{
  "text": "Proxmox Notification:\nSeverity: {{ escape severity }}\n\nTitle: {{ escape title }}\n\nMessage: {{ escape message }}"
}
```

**Benefits:**
- Clean, readable format
- Telegram API properly accepts JSON POST requests
- `{{ escape }}` handles special characters automatically
- Easier to modify message format

**Validation method:** Temporarily disabled NAS storage mount, forced backup failure, notification arrived correctly

**Tutorial reference:** https://forum.proxmox.com/threads/problem-with-webhook-notification-targets.162839/

---

## Home Assistant → Telegram Setup

### YAML Configuration Attempt (Hours of Troubleshooting)

**Initial approach:** Configure via `configuration.yaml`
- Created bot via BotFather
- Obtained personal chat ID and group chat IDs
- Added `telegram_bot` and `notify` platforms to YAML

**Issues encountered:**

**Supergroup Migration:**
- Regular Telegram groups upgraded to "supergroups" when bot added with permissions
- Chat IDs changed format (original → `-100xxxxxxxxxx` supergroup format)
- Initial config used old IDs → notifications failed

**Bot Permissions:**
- Groups showed "has no access to messages" until bot made admin
- Required admin status with "Post messages" permission enabled
- Other admin permissions (delete, ban, invite) disabled for security

**YAML vs UI Conflict:**
- HA showed deprecation warning: "Telegram Bot YAML configuration is being removed"
- Cannot have both YAML and UI configuration simultaneously
- Warning message unclear about the conflict

**Troubleshooting challenges:**
- Error messages generic ("unauthorized", "invalid chat IDs")
- Multiple config iterations with different chat ID combinations
- Hours spent debugging unclear errors

### UI Configuration Solution (10 Minutes)

**Switch to UI:**
- Removed entire Telegram configuration from `configuration.yaml` (both `telegram_bot` and `notify` sections)
- Restarted HA
- Telegram Bot integration appeared in Devices & Services
- Added allowed chat IDs via UI

**Supergroup issue still present:**
- Initial chat IDs failed
- Logs showed explicit error: `Group migrated to supergroup. New ID: -100xxxxxxxxxx`
- Updated to new supergroup IDs (format: `-100xxxxxxxxxx`)
- Integration successful

**Result:** ✅ `notify.send_message` service operational

**Why UI was easier:**
- Clearer error messages with exact new chat IDs in logs (vs generic YAML errors)
- Faster iteration (UI updates vs config file editing + restart cycle)
- Total: YAML hours vs UI 10 minutes

### Notification Groups Created

**Critical Alerts:** System health, backup failures, automation failures, device offline  
**Info Notifications:** Backup success, routine automations, informational status

**Bot admin permissions:**
- ✅ Post messages (only this enabled)
- ❌ All other permissions disabled for security

### Proof of Concept Validation

**System Monitor integration installed**

**Test automation:**
- Trigger: Memory usage > 20%
- Action: Send Telegram notification

**Result:** ✅ Notification received, HA → Telegram validated

**Node-RED evaluation:**
- Attempted for test automation
- Too complex for simple notification needs
- HA native automation sufficient
- Deferred: Re-evaluation in S12 if complex logic needed

### Outstanding HA Notification Work

**Deferred to S12:**
- Comprehensive notification strategy (what/when/how)
- Dynamic messages with entity values
- System health thresholds
- Device offline detection
- Backup status monitoring
- Automation failure tracking

**Reason:** Implementation scope too large for S9, requires methodical planning

---

## ZWave Stick Firmware Update

**Issue identified:** Firmware 7.17.1 had early 700 series bug (mesh flooding, controller unresponsive), resolved in SDK 7.17.2+

**Warning message in HA:** "Early 700 series firmware revisions had a bug that could cause the mesh to be flooded on some networks and the controller to become unresponsive"

**Initial troubleshooting attempts:**
1. Uninstalled Z-Wave JS addon
2. Installed Z-Wave JS UI addon (thought it would provide firmware update capability)
3. Reconfigured device
4. Z-Wave JS UI showed "up to date" but still on 7.17.1

**Discovery:** HA addons cannot update ZWave stick firmware - addon manages ZWave network, not stick hardware itself

**Solution required:** Manufacturer-specific external tool

**Zooz ZST10 700 series update process:**
- Tool: Simplicity Studio (Silicon Labs)
- Tutorial: https://www.support.getzooz.com/kb/article/931-how-to-perform-an-ota-firmware-update-on-your-zst10-700-z-wave-stick/
- Platform: Windows laptop required (not Linux/Proxmox direct)

**Note:** Firmware update process is manufacturer-specific - Zooz uses Simplicity Studio, other ZWave stick brands (Aeotec, Nortek, Z-Wave.Me) use different tools and processes

### Proxmox USB Passthrough Configuration Change

**Before migration:**
- Changed USB passthrough from "Use USB Port" to "Use USB Vendor/Device ID"
- Reason: Device ID follows stick regardless of physical USB port, more stable for firmware update workflow (unplug → update → replug)

**Process:**
1. Proxmox: VM Hardware → USB Device → Edit
2. Selected "Use USB Vendor/Device ID"
3. Chose: Zooz_ZST10_700_Z-Wave_Stick (10c4:ea60)

**Benefit:** Stick recognized immediately after reconnecting post-firmware-update

### Firmware Update Execution

**Process:**
1. Shut down Proxmox
2. Removed ZWave USB stick
3. Connected to Windows laptop
4. Ran Simplicity Studio

**Simplicity Studio experience:**
- Very slow performance
- Instructions unclear
- User uncertainty: uninstalled/reinstalled application, still unsure if process would work
- Ultimately successful despite poor UX

**Result:**
- Firmware: 7.17.1 → 7.18.3
- Warning message disappeared in HA
- Stick reconnected to Proxmox automatically (Device ID passthrough validated)
- ZWave integration remained functional

---

## ZWave Device Migration (Phase 2)

**Devices migrated:** 2 ZEN71 On/Off switches
1. Tenant Side Porch Light (outdoor)
2. Side Light (outdoor)

**Migration rationale:**
- SmartThings bridge reliability issues (Tenant Side Door Light consistently failed in automation - 4 of 5 devices succeeded, 1 consistently dropped)
- SmartThings entity exposure limitations (basic on/off only, no dimmer controls, no manufacturer/firmware info)
- Test direct ZWave control before committing to full migration of 33+ remaining switches

### First Switch Migration (Tenant Side Porch Light)

**Process:**
1. SmartThings app: Exclude device from network
2. HA Z-Wave JS: Add node (inclusion mode)
3. Physical switch: Tap UP paddle 3x quickly
4. QR code scanned for S2 security (faster than manual PIN entry)
5. Device interview completed automatically
6. OTA firmware update initiated and completed

**Result:** ✅ Operational, firmware updated (capability not available via SmartThings)

### Second Switch Migration (Side Light)

**Process:**
1. Excluded from SmartThings
2. Included in HA Z-Wave JS
3. QR code scanned

**Issue:** Interview stuck at "This device is currently being interviewed and may not be fully operational"

**Troubleshooting:**
- Waited 10+ minutes: No progress
- Tapped paddle multiple times: No effect
- Excluded device and re-included

**Re-inclusion without QR code attempted:**
- Chose "other method" during inclusion
- Prompted for S2 security PIN (5-digit code printed on device label)
- PIN unknown, process couldn't continue

**Resolution:** Unscrewed faceplate, rescanned QR code, successful inclusion

**Lesson learned:** Photographed both switch QR codes for future reference (avoids unscrewing faceplate for re-pairing)

**S2 Security explanation:** ZWave S2 uses QR code (contains Device Specific Key) OR manual PIN entry. QR code bypasses PIN requirement and is faster method.

**Result:** ✅ Both switches operational, firmware updates completed

### Entity Exposure Comparison

**SmartThings bridge:**
- Basic on/off control only
- Can manually dim in HA UI (slider works)
- NO dimmer level entity for programmatic control
- Cannot check brightness in automations
- Cannot set specific brightness values
- Missing: manufacturer info, firmware version, device model, power monitoring

**Direct ZWave integration:**
- Full device control exposure (validation pending deeper testing)
- OTA firmware update capability
- Device metadata visible
- Advanced configuration options available

### Automation Recreation

**Automation:** "Outside Lights On"
- Trigger: Sunrise with offset
- Target: Both migrated switches + 2 TPLink Kasa switches
- Created in HA (previously ran in SmartThings app as workaround for reliability issues)

**Validation test:** Morning sunrise automation
- ✅ Both ZWave switches responded
- ✅ All devices turned on (no command drops)
- ✅ Automation completed successfully

**Comparison to SmartThings bridge behavior:**
- Previous: 5 devices attempted, 4 succeeded, Tenant Side Door consistently failed
- Direct ZWave: 4 devices attempted, 4 succeeded (includes 2 ZWave + 2 via TPLink integration)

**Conclusion:** Direct ZWave control eliminates timing-dependent failures observed with SmartThings cloud bridge

---

## Honeywell T5 Integration Resolution

**Background:** OAuth approval pending since S8 (submitted expected approval within 2 weeks, still no response over month)

**Alternative approach discovered:** HomeKit integration

**Process:**
1. Thermostat: Settings → Menu → Reset → HomeKit
2. Pairing code displayed on thermostat screen
3. HA: Settings → Devices & Services → HomeKit integration
4. Added device with pairing code

**Result:** ✅ Operational, local control

**Entity exposure via HomeKit:**
- Temperature setpoint
- On/Off
- Current temperature sensor  
- Unit selection (Fahrenheit/Celsius)

**Missing controls:**
- Hold settings
- Schedule configuration
- Mode changes beyond basic on/off
- Advanced thermostat features

**Comparison to Ecobee HomeKit integration:**
- Ecobee: Full control (mode/hold/scheduling) via HomeKit
- Honeywell: Basic control only

**User assessment:**
- Acceptable for current use case (one-room heat thermostat, light use)
- Not ideal (missing schedule/hold management capabilities)
- Monitoring real-world usage to determine if limitation becomes problematic

**Future consideration:** Replace thermostat (different brand or ZWave version with better HA integration) if basic control proves insufficient. Not priority for single-room, light-use application.

**Status change:** S8 Outstanding item "Honeywell T5 OAuth approval" → Resolved via HomeKit alternative (OAuth path abandoned)

---

## NAS Hardware Issue (Ongoing, Not Resolved)

**Background from previous sessions:** Synology DS218j with 2x drives in RAID 0, Drive 2 failing intermittently causing periodic volume crashes

**Current failure pattern:**
- Drive 2 crashes every ~1 week
- Entire volume becomes read-only (both Drive 1 and Drive 2 despite Drive 1 being healthy)
- System restart required to restore write access
- Cycle repeats after ~1 week

**Real-world notification validation:** During S9 work, NAS crashed and Telegram notification received ✅ (confirms Synology → Telegram integration functional in production)

**RAID 0 architecture:**
- Data striped across both drives (split for performance)
- Zero redundancy
- Either drive failure = complete data loss
- Cannot remove Drive 2 without losing all data

**Current backup strategy:**
- iDrive cloud backup (offsite redundancy)
- No local backup currently

**Migration strategy discussed:**
- New 6TB drive purchased
- Multiple approaches evaluated (RAID 1, JBOD, Basic volume)
- Restoration options: iDrive cloud restore vs local backup
- Decision deferred - separate from S9 scope

**Plan:** Address in S10 before S11 full ZWave migration (avoid migrating 33+ switches without working backup)

**Critical validation from this incident:** Drive crashes don't destabilize HA with Samba Backup addon (confirms S6 architecture decision to avoid network storage mount dependencies)

---

## Key Learnings

**Technical:**
- Supergroup migration not handled automatically in telegram config (UI integration detected issue and clearly indicated chat ID changes)
- HA addons manage ZWave network, cannot update stick firmware (manufacturer tools required)
- ZWave stick firmware update process is manufacturer-specific (Zooz → Simplicity Studio, other brands use different tools)
- USB passthrough by Device ID more stable than port-based (survives physical port changes)
- S2 security QR code bypasses PIN requirement (faster inclusion, photograph for future use)
- Direct ZWave control eliminates SmartThings cloud bridge timing failures
- Iterative device migration maintains system functionality (test per-device vs big bang cutover)
- Node-RED complexity not justified for simple single-condition notifications (HA native automation sufficient)

**Project Management:**
- Platform evaluation requires threat model assessment (security trade-offs documented explicitly)
- UI configuration can handle edge cases better than manual YAML (supergroup migration example)
- Tool UX quality doesn't determine functionality (Simplicity Studio slow/unclear but worked)
- Real-world failure testing validates notification systems better than synthetic tests
- Troubleshooting time can exceed implementation time (YAML hours vs UI 10 minutes)
- Proof of concept sufficient for infrastructure validation (comprehensive implementation deferred when scope too large)

---

## Resources

- [Synology DSM 7.1 Telegram Guide](https://medium.com/@nikolayburyak/synology-dsm-7-1-telegram-alerts-b9c27fcbf8de)
- [Proxmox Telegram Webhook Configuration](https://forum.proxmox.com/threads/problem-with-webhook-notification-targets.162839/)
- [Zooz ZST10 Firmware Update Guide](https://www.support.getzooz.com/kb/article/931-how-to-perform-an-ota-firmware-update-on-your-zst10-700-z-wave-stick/)

---

## Next Session (S10 Planned)

**External Access + Google Assistant Integration + NAS Replacement**
- Expose HA externally (Nabu Casa Cloud vs self-hosted options)
- Google Assistant integration for voice control
- NAS Drive 2 replacement and iDrive restore validation
- Decision point: If iDrive restore too slow, assess risk vs waiting before S11 migration

---

**End Session 9**
