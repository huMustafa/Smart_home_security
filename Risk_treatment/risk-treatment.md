# Task 5: Risk Treatment and Residual Risk

For each High-risk threat, I've decided how to handle it and why. I've also noted where risks remain.


## Risk Treatment Table

| # | Threat | Treatment | Rationale |
|---|---|---|---|
| T1 | Device identity spoofing | **Mitigate** — mTLS with device certificates | Directly solvable at the architecture level. Cost of implementing a device CA is justified for a lock system. |
| T2 | MitM command tampering | **Mitigate** — TLS 1.3 + message-level signing for lock commands | TLS is table stakes. For the lock command specifically, adding a message-level HMAC is worth the overhead. |
| T4 | Camera footage interception | **Mitigate** — end-to-end encryption + AES-256 at rest | The sensitivity of the data makes this non-negotiable. This is the one control I'd fight hardest to keep if someone tried to cut it for cost. |
| T8 | Lateral movement from compromised device | **Mitigate** — VLAN segmentation + per-device MQTT authorization | Segmentation limits the blast radius. You can't fully prevent a device from being compromised, but you can stop the damage from spreading. |
| T9 | Admin portal exposed to internet | **Avoid** — move admin portal behind VPN, off public internet | There's no good reason for the admin portal to be internet-facing. This isn't a mitigation, it's just removing the risk by design. |
| T10 | Tampered firmware update | **Mitigate** — signed firmware, offline signing keys | Non-negotiable for a fleet of physical devices. Compromised firmware is persistent and hard to recover from. |


## Medium Risk Treatment

| # | Threat | Treatment | Rationale |
|---|---|---|---|
| T3 | Repudiation of commands | **Mitigate** — immutable audit logs | Relatively cheap to implement, high forensic value |
| T5 | Behavioral profiling | **Mitigate** — row-level auth, UUIDs, data minimization | IDOR is solvable with good API design |
| T6 | Hub DoS from LAN | **Accept** — rate limiting helps, but limited mitigation possible | Requires physical LAN access. Rate limiting the MQTT broker reduces severity. Full mitigation would require hardware-level upgrades. |
| T7 | API backend DDoS | **Transfer** — handled by API gateway / CDN layer | A managed API gateway with built-in rate limiting and DDoS protection is the standard approach. Not cost-effective to build this ourselves. |
| T11 | Over-permissioned OAuth | **Mitigate** — minimal OAuth scopes, device-type restrictions | Simple to implement at the authorization server level. |
| T12 | Log tampering | **Mitigate** — append-only log store | Write-once log storage is a standard pattern. |


## Residual Risks

### Accepted Residual Risk: Physical Device Tampering

Even with mTLS and signed firmware, an attacker with sustained physical access to a device (e.g., they took the door lock off the door) can attempt to extract private keys from flash memory or jtag the device. Fully preventing this would require hardware security modules (HSMs) embedded in every device, which isn't realistic at consumer price points.

**Why it's accepted:** The threat model for this scenario assumes the attacker already has physical access to the home. At that point, the adversary model has changed — this is more of a forensics problem than a live security problem. It's worth acknowledging, but not worth redesigning the system around.

### Accepted Residual Risk: Zero-Day Vulnerabilities in Third-Party Integrations

The Alexa and Google Home integrations depend on those platforms' security. If Google Home has a zero-day in their OAuth implementation, there's not much the smart home system can do about it beyond minimal scoping and quick response. This risk is accepted because it's outside our control, and the mitigation (minimal OAuth scopes) is already in place.

### Accepted Residual Risk: Social Engineering of Admin Staff

The admin portal is behind VPN and MFA, but a sophisticated attacker targeting a specific admin via phishing or SIM swapping could still compromise an admin account. Security awareness training and strict account recovery procedures help, but they don't eliminate the risk. Accepted because it's an organizational control problem, not an architectural one.
