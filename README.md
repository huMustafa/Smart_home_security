# Secure Architecture & Design
## Smart Home IoT System — Security Analysis

**Scenario:** Option F — Any Organization  
**System:** A smart home IoT ecosystem  
**Author:** Muhammad Mustafa 


## What This Is

This repo is my submission for the Secure Architecture & Design assignment. I chose a smart home IoT system because it's a genuinely interesting case, it mixes consumer-facing software, embedded hardware, cloud backends, and third-party integrations, all in a context where the physical consequences of a security failure (someone unlocking your front door remotely) are very real.

The analysis covers system architecture, asset identification, STRIDE-based threat modeling, architectural security controls, and risk treatment. All diagrams were drawn in Mermaid and are in the `/Architecture` folder.

## File Index

| File | Task | Contents |
|---|---|---|
| `architecture/system-definition.md` | Task 1 | System components, data flows, trust boundaries, diagram description |
| `architecture/diagram-v1.png` | Task 1 | Original architecture diagram |
| `architecture/diagram-v2.png` | Task 4 | Updated architecture with security controls |
| `assets/asset-inventory.md` | Task 2 | Asset inventory table mapped to CIA + Accountability |
| `threat-model/stride-table.md` | Task 3 | Full STRIDE threat model with scenario explanations |
| `controls/security-controls.md` | Task 4 | Architectural security controls with justifications |
| `risk-treatment/risk-table.md` | Task 5 | Risk treatment decisions + residual risk explanation |
| `report/full-report.md` | Task 6 | Combined final report tying all tasks together |


## Quick Summary

- **Threats identified:** 12 (across all STRIDE categories)
- **High-risk threats:** 5
- **Primary threat modeling framework:** STRIDE
- **Key controls:** mTLS device auth, VLAN segmentation, end-to-end encryption, immutable audit logs, signed OTA firmware updates
- **One accepted residual risk:** Physical device tampering with direct hardware access

