# Task 3: Threat Model (STRIDE)

I used the STRIDE framework for this. STRIDE stands for Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privilege. I went through each component of the architecture and asked "what could go wrong here?" from each STRIDE angle. Some components had multiple applicable threats,  I've tried to keep the table focused on the most realistic and impactful ones rather than listing every theoretical possibility.

---

## STRIDE Threat Table

| # | Threat | STRIDE | Affected Component | Impact | Risk Level |
|---|---|---|---|---|---|
| T1 | Attacker spoofs a smart device identity on the LAN | Spoofing | IoT Hub ↔ Device | Hub accepts commands from malicious device | **High** |
| T2 | MitM attack modifies a lock/unlock command in transit | Tampering | API Backend ↔ IoT Hub | Door opened without user intent | **High** |
| T3 | User denies having issued a "disarm alarm" command | Repudiation | Logging / Audit System | No accountability for security-critical actions | Medium |
| T4 | Camera footage intercepted during cloud upload | Information Disclosure | Device ↔ Cloud Storage | Private video footage exposed to attacker | **High** |
| T5 | Behavioral data inferred from device event logs | Information Disclosure | Cloud Storage / API | User's home/away patterns leaked | Medium |
| T6 | Flood of fake device events crashes the IoT Hub | Denial of Service | IoT Hub | Local automation fails, devices unresponsive | Medium |
| T7 | DDoS against API backend locks users out of app | Denial of Service | API Gateway / Backend | Users can't control devices remotely | Medium |
| T8 | Compromised smart bulb escalates to hub admin access | Elevation of Privilege | IoT Hub ↔ Device Fleet | Full hub compromise from a low-consequence device | **High** |
| T9 | Admin portal accessible from public internet | Elevation of Privilege | Web Admin Portal | Attacker gains fleet-wide admin access | **High** |
| T10 | Tampered firmware image pushed to door lock | Tampering | Firmware Update Server ↔ Hub ↔ Devices | Backdoored firmware enables persistent access | **High** |
| T11 | Alexa integration over-permissioned via OAuth | Elevation of Privilege | Third-Party Integration ↔ API | Voice assistant can issue commands beyond scope | Medium |
| T12 | Log tampering to hide unauthorized access | Tampering | Audit Logs | Attacker erases evidence of their activity | Medium |

---

## Threat Scenario Explanations

### T1 — Device Spoofing (Spoofing | High)
An attacker who gains access to the home WiFi network (not that hard — maybe they got the guest password, or cracked WPA2) can enumerate the local network and identify connected IoT devices by their MAC addresses or MQTT topic names. If the hub authenticates devices only by MAC address or a shared pre-shared key, the attacker can impersonate a legitimate device — say, the front door lock — and inject fake status messages. The hub now thinks the lock is closed when it isn't, or processes fake "lock confirmed" acknowledgments. The physical consequence is obvious.

### T2 — Command Tampering in Transit (Tampering | High)
If the communication channel between the API backend and the IoT hub isn't using mutual TLS (just regular TLS or, worse, plain HTTP), an attacker positioned between the two — either through a rogue WiFi access point or a compromised home router — can intercept and modify command packets. A "lock door" command could be flipped to "unlock door" by changing a single byte. This is especially concerning because the hub often processes commands without a secondary confirmation step, so there's no human checkpoint before the actuator fires.

### T3 — Repudiation of Security Commands (Repudiation | Medium)
Imagine a scenario where someone arms/disarms the alarm and then denies doing it — either a family dispute situation or a more serious case where a compromised account was used. If the system doesn't log commands with enough detail (user ID, timestamp, session token, source IP), there's no way to prove what happened. This is categorized Medium because the direct harm is limited, but in an insurance claim or police investigation scenario, missing logs are a real problem.

### T4 — Camera Footage Interception (Information Disclosure | High)
Camera footage is continuously uploaded to cloud storage for recording and remote viewing. If this upload isn't encrypted end-to-end — or if it's encrypted in transit but the encryption keys are shared/weak — an attacker monitoring the network or with read access to cloud storage buckets can view live or recorded footage from inside the home. The reason this is High rather than Medium is that IP cameras are often placed inside bedrooms, nurseries, or living areas. This is a serious privacy violation with potential for blackmail or enabling physical burglary.

### T5 — Behavioral Profiling from Device Logs (Information Disclosure | Medium)
Even without accessing camera footage, the pattern of device events tells a story: lights turning on at 7am, thermostat kicking in at 6:30am, door lock disengaged at 8:15am, same pattern reversed in the evening. This is a pretty accurate map of when the home is occupied. If the API exposes event logs without proper authorization checks (e.g., an IDOR vulnerability where you just increment a user ID), an attacker can build a "home/away" schedule for a target. Combined with physical reconnaissance, this meaningfully increases burglary risk.

### T6 — IoT Hub Denial of Service (Denial of Service | Medium)
An attacker on the local network (or a compromised device on the network) can flood the hub's MQTT broker with a high volume of fake device events or connection requests. The hub, running on embedded hardware with limited RAM and CPU, doesn't have the same resilience as a cloud server. If the hub crashes or becomes overwhelmed, local automation fails entirely — lights don't respond, alarm systems don't trigger, locks might default to an unsafe state depending on configuration. Risk is Medium because the attack requires LAN access and doesn't directly cause a breach.

### T7 — API Backend DDoS (Denial of Service | Medium)
The API backend is internet-facing, which means it's susceptible to volumetric DDoS attacks. An attacker could knock the backend offline, preventing users from remotely controlling their devices via the mobile app. This is Medium rather than High because local automation through the hub should still work, and most smart home systems are designed to degrade gracefully. But if someone is locked out of their house and the app is down, or if the alarm can't be remotely disarmed, it becomes a real availability problem.

### T8 — Lateral Movement from Compromised Device (Elevation of Privilege | High)
This is one of the trickier threats. A smart bulb or low-end IoT sensor probably has weak firmware, minimal security patches, and might be running with default credentials. If an attacker compromises a bulb, they're now on the internal IoT network. If all devices on that network share the same MQTT topic space or the hub doesn't enforce per-device authorization, the compromised bulb can publish to topics it shouldn't — including the door lock's command topic or the hub's admin configuration interface. It's the IoT equivalent of pivoting from a printer to a domain controller.

### T9 — Admin Portal Exposed to Public Internet (Elevation of Privilege | High)
If the admin portal is accessible at a public IP without VPN, an attacker doesn't even need to compromise anything inside the network — they can go straight for the admin interface. Credential stuffing, brute force, or exploiting an unpatched vulnerability in the admin web app could give full platform access. From there, the attacker can push malicious firmware to all devices on the platform, read all user data, or disable accounts. This is High because it's a single point of failure that affects all customers, not just one household.

### T10 — Tampered Firmware Update (Tampering | High)
Firmware updates get pushed to devices through the hub. If the update server doesn't sign firmware packages, or if the hub doesn't verify signatures before installing, an attacker who can intercept or spoof the update delivery (either via DNS hijacking or a compromised update server) can push malicious firmware to every device in the fleet. A backdoored door lock firmware that silently accepts a hardcoded unlock code is a nightmare scenario. The damage is persistent — even after a "fix," devices would need to be factory reset or re-flashed.

### T11 — Over-Permissioned Third-Party OAuth (Elevation of Privilege | Medium)
When linking Alexa or Google Home, the OAuth flow grants permissions to those services. If the scope isn't tightly defined (e.g., granting `device:write:all` when you only need `device:write:lights`), the voice assistant integration can issue commands to devices it shouldn't be able to control — like unlocking the front door via a voice command that anyone in earshot can trigger. This is particularly uncomfortable because the attack surface is literally your living room.

### T12 — Log Tampering (Tampering | Medium)
If an attacker gains write access to the logging system — or if log files aren't protected with integrity controls — they can alter or delete evidence of their activity. This is a post-exploitation concern rather than an initial attack vector, but it significantly increases the difficulty of forensic investigation. Logs that can be modified aren't really audit logs.

---

## Risk Summary

| Risk Level | Count | Threats |
|---|---|---|
| High | 5 | T1, T2, T4, T8, T9, T10 |
| Medium | 6 | T3, T5, T6, T7, T11, T12 |
| Low | 0 | — |

I didn't assign any Low risks — my reasoning is that if something made it into this table, it's at least Medium. Threats that genuinely seemed Low I just didn't include.
