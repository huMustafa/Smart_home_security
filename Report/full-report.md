# CS 382 — Secure Architecture & Design
# Final Report: Smart Home IoT Security Analysis

**Author:** Muhammad Mustafa  

 

## 1. System Overview

The system modeled in this assignment is a smart home IoT ecosystem — specifically, the kind of connected home platform that a company like Ring, SmartThings, or a hypothetical competitor might deploy. The system lets homeowners control smart locks, IP cameras, lights, and thermostats through a mobile app, while also supporting voice control through Alexa and Google Home.

This system is interesting from a security perspective because it sits at the intersection of consumer software, cloud infrastructure, embedded hardware, and physical security. A vulnerability that might be Medium severity in a web app becomes High severity when the affected component is a front door lock.

The scenario fits under **Option F (Any Organization)** — treating the smart home platform as an organization with assets, users, and a threat surface that spans multiple layers.


## 2. Architecture Diagram

See `Architecture/diagram-v1.png` for the original architecture and `Architecture/diagra,-v2.png` for the updated architecture with security controls applied.

**v1 components:** Mobile app, API gateway, auth/device/notification microservices, IoT hub, device fleet (locks, cameras, lights), cloud storage, admin portal, third-party integrations (Alexa, Google Home), firmware update server.

**v2 additions:** WAF + rate limiter on API gateway, VPN gateway in front of admin portal, VLAN isolation around IoT hub and device fleet, secrets manager connected to backend services, append-only SIEM/audit log store, device CA for certificate issuance, TLS 1.3 labels on all in-transit paths, AES-256 labels on storage, signed OTA labels on firmware path.


## 3. Asset Inventory

*(Full table in `asset-inventory.md`)*

The 12 assets identified span credentials, behavioral data, device commands, firmware, API keys, audit logs, hardware (hub and devices), admin access, automation rules, and OAuth tokens.

The most asymmetric asset is **device commands (A3)**: confidentiality is relatively low importance, but integrity and availability are critical — a dropped or modified lock command has immediate physical consequences.


## 4. Threat Model

*(Full STRIDE table and scenario explanations in `threat-model.md`)*

STRIDE was applied across all major components and data flows. 12 threats were identified:

- **5 High-risk:** Device spoofing (T1), command tampering (T2), camera footage interception (T4), lateral movement from compromised device (T8), internet-exposed admin portal (T9), tampered firmware update (T10)
- **6 Medium-risk:** Command repudiation (T3), behavioral profiling (T5), hub DoS (T6), API DDoS (T7), over-permissioned OAuth (T11), log tampering (T12)

The most realistic high-risk attack chain would be: compromise a low-security device (smart bulb) via default credentials → pivot within the IoT VLAN to the hub → escalate to hub admin → issue commands to the door lock. This chain is addressed by controls in Task 4.


## 5. Security Controls

*(Full justifications in `security-controls.md`)*

Controls are organized by category:

| Category | Key Control |
|---|---|
| Identity & Access Management | mTLS device auth, OAuth 2.0 + PKCE, hardware MFA for admin |
| Network Segmentation | IoT VLAN isolation, admin portal behind VPN, separate API paths |
| Data Protection | TLS 1.3 in transit, AES-256 at rest, UUID-based IDs |
| Secrets Management | HashiCorp Vault for API keys, minimal OAuth scopes |
| Monitoring & Logging | Append-only audit logs, behavioral anomaly detection, SIEM |
| Secure Deployment | Signed firmware (offline keys), unique device defaults, port hardening |

The highest-value control in this system is probably **VLAN segmentation** — it's the one that limits blast radius most effectively and doesn't require changes to the devices themselves (which you often can't update on short notice).


## 6. Residual Risks

*(Full treatment table in `risk-treatment.md`)*

Three residual risks are accepted:

1. **Physical device tampering** — HSMs would address this but aren't cost-justified at consumer price points.
2. **Zero-days in third-party integrations** — outside our control; already mitigated at the boundary with minimal OAuth scopes.
3. **Social engineering of admin staff** — an organizational training problem, not an architectural one.


## 7. Assumptions and Limitations

- This model assumes the IoT hub communicates with the cloud backend over a standard home internet connection (broadband). It doesn't model satellite or cellular backup scenarios.
- Zigbee/Z-Wave RF communication between devices and the hub is mentioned but not threat modeled in depth — RF-level attacks (replay, jamming) are real but would benefit from their own analysis.
- The model assumes the platform provider is a single organization. In a white-label deployment scenario (where a third party hosts the platform), the trust model changes significantly.
- Physical security of the hub device itself is partially addressed (it should be secured inside the home) but a full physical security analysis is out of scope here.
- Threat likelihoods are estimated without real breach data — risk levels are based on reasonable attacker motivation and known IoT attack patterns, not historical incident rates.


## 8. References and Frameworks Used

- STRIDE Threat Modeling Framework — Microsoft
- OWASP Threat Modeling Process — owasp.org
- IoT Security Architecture: Trust Zones and Boundaries — build5nines.com
- Smart Home Threat Model reference — github.com/kkredit/smart-home-threat-model
- HashiCorp Vault — for secrets management reference
- Mermaid — for architecture diagrams
