# Session 1: 2025/08/25 - Proxmox Installation & Network Discovery
**Status:** 🔄 In Progress  
**Time Invested:** 2 hours

## Planned vs Actual:
| Task | Original Plan | Actual Status | Notes |
|------|---------------|---------------|-------|
| Backup Surface Book 2 data | Day 1 | ✅ Complete | |
| Windows license extraction | Day 1 | ✅ Complete | |
| Link MS account | Day 1 | ✅ Complete | |
| Document BIOS settings | Day 2 | ✅ Complete | |
| Create Proxmox USB installer | Day 2 | ✅ Complete | Used Etcher to flash ISO |
| Test Proxmox live boot | Day 3 | ✅ Complete | |
| Full Proxmox installation | Day 4 | 🔄 Partial | Install complete, network config pending |

## Key Discovery: WiFi Installation Blocker
- **Issue:** Proxmox installation cannot connect to WiFi during initial setup
- **Impact:** Unable to access web interface for configuration
- **Research:** Video tutorial used instead of written guides ([YouTube reference](https://www.youtube.com/watch?v=zngSuqCM4d8))
- **Solution:** USB ethernet adapter required for network connectivity

## Hardware Decision: Ethernet + USB Strategy
**Decision:** Start with budget ethernet adapter approach
- **Rationale:** Standard virtual networking (not passthrough) reduces compatibility requirements
- **Target:** USB 3.0 to Gigabit adapter with Realtek RTL8153 chipset (~$15-20)
- **Future scope:** Use existing USB hubs for Z-Wave dongle passthrough testing

## Current System State
- Proxmox installation: ✅ Complete
- Web interface access: ❌ Blocked by network
- Next blocker: Ethernet adapter procurement and network configuration

## Timeline Adjustment
**New approach:** Session-based tracking instead of daily milestones
- **Target:** ~6 hours/week (2 hours × 3 sessions)
- **Realistic range:** 4-8 hours/week depending on schedule

## Community Resources Used:
- [Proxmox Installation Video Tutorial](https://www.youtube.com/watch?v=zngSuqCM4d8) - More helpful than written guides

## Next Session Focus:
- [ ] Purchase USB ethernet adapter (Realtek RTL8153 chipset)
- [ ] Configure Proxmox networking via web interface
- [ ] Document network configuration for future reference
- [ ] Begin Home Assistant VM creation

## Key Learnings:
- WiFi during Proxmox installation is not practical/possible
- Video tutorials more effective than documentation for initial setup
- Hardware validation phase is necessary before proceeding
- Session-based timeline more realistic than daily milestones

## Network Architecture Clarification:
- **Ethernet:** Standard virtual networking through Proxmox bridge
- **USB passthrough:** Only needed for Z-Wave dongle (Zooz), not ethernet
- **Future devices:** Zigbee/Matter dongles will require passthrough research
