# homelab-project-management
## Project Scope
Deploy a self-hosted home automation system using Proxmox as virtualization platform, running Home Assistant to manage smart home devices, with focus on security, reliability, and scalability for future additions. Secondary VM planned for Reolink/Wyze camera display system.
Success Criteria

Proxmox running stable on repurposed Surface Book 2 laptop
Home Assistant VM successfully deployed and accessible via web interface
At least 3 different device types integrated (e.g., lights, thermostat, sensors)
Automated backup solution configured
Local network access secured with HTTPS
System maintains 99% uptime over 30-day period
Windows 10 license preserved for future VM use

## Milestone Timeline

Week 1: Research hardware requirements, backup laptop data, install Proxmox
Week 2: Configure Proxmox networking, create Home Assistant VM
Week 3: Initial Home Assistant setup, integrate first devices
Week 4: Add remaining devices, configure automations
Week 5: Implement backup strategy, security hardening
Week 6: Documentation, performance optimization, lessons learned

## Success Criteria

## Hardware Requirement
ComponentCurrent StateFuture NeedPriorityZ-Wave✅ USB dongle owned-CompleteZigbee❌ Not ownedUSB dongle requiredHigh - Week 3Matter❌ Not ownedUSB dongle (optional)Low - Month 2USB Expansion✅ Hub available-CompleteNetworkWiFi (built-in)USB ethernet adapterMedium - Week 2


## Risk Register

## Decisions Log
2025-07: Hardware Platform Selection

Decision: Use Surface Book 2 laptop instead of Raspberry Pi for Home Assistant
Rationale: Community feedback indicates Pi has stability issues with HA; existing Surface Book 2 available at no cost
Trade-offs: Surface may have compatibility issues with Proxmox, but web GUI management makes peripheral issues acceptable

2025-07: Virtualization Platform - Proxmox VE

Decision: Selected Proxmox as hypervisor
Rationale: Zero licensing cost, proven HA compatibility based on community deployments, allows future VM expansion
Future scope: Second VM planned for Reolink/Wyze camera display system

2025-07: Network Architecture

Decision: Start with WiFi, migrate to wired later
Rationale: Prove feasibility before purchasing USB network adapter
Success criteria: If stable on WiFi for 7 days, purchase dedicated ethernet adapter

2025-07: Windows 10 License Preservation Strategy

Decision: Keep Windows 10 (not upgrade to 11), link to Microsoft account before Proxmox installation
Rationale: Digital license ties to MS account, improves reactivation chances in VM; Win10 better for older camera software
Risk: No guaranteed method for OEM license transfer to VM
Approach: Document multiple extraction methods, include phone activation as fallback

## Lessons Learned
