# Task 2: Asset Identification and Security Objectives

## Asset Inventory

| # | Asset | Type | Confidentiality | Integrity | Availability | Accountability |
|---|---|---|---|---|---|---|
| A1 | User credentials (passwords, tokens) | Data | **Critical** | **Critical** | Medium | **Critical** |
| A2 | Camera footage (live + recorded) | Data | **Critical** | High | High | High |
| A3 | Device command stream (lock/unlock, arm/disarm) | Data | Medium | **Critical** | **Critical** | **Critical** |
| A4 | Personally identifiable / behavioral data | Data | **Critical** | High | Medium | High |
| A5 | Firmware images | Software | Low | **Critical** | High | High |
| A6 | API keys and service credentials | Data | **Critical** | **Critical** | Medium | **Critical** |
| A7 | Audit and event logs | Data | Medium | **Critical** | High | **Critical** |
| A8 | IoT Hub (physical device) | Hardware | N/A | **Critical** | **Critical** | High |
| A9 | Device fleet (locks, cameras, lights) | Hardware | N/A | **Critical** | High | Medium |
| A10 | Admin portal access | System | **Critical** | **Critical** | High | **Critical** |
| A11 | Automation rules and schedules | Data | High | **Critical** | Medium | Medium |
| A12 | OAuth tokens (third-party integrations) | Data | **Critical** | High | Medium | High |

---

## Notes

- **A3 (device commands)** is the one I'd argue is the most asymmetric,  confidentiality matters less than you'd think (knowing that a lock was unlocked isn't the end of the world), but integrity and availability are critical because a tampered or dropped "lock" command has immediate physical consequences.
- **A5 (firmware)** has low confidentiality requirements,  firmware is often public anyway,  but a tampered firmware image pushed to a door lock is a catastrophic integrity failure.
- **A8 (IoT Hub)** is interesting because it's hardware sitting in someone's home, so physical security is a real concern that you don't usually have to think about with cloud systems.
