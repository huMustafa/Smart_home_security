# Secure Architecture & Design
## Smart Home IoT System — Security Analysis

**Scenario:** Option F — Any Organization  
**System:** A smart home IoT ecosystem  
**Author:** Muhammad Mustafa 


## What This Is

This repo is my submission for the Secure Architecture & Design assignment. I chose a smart home IoT system because it's a genuinely interesting case, it mixes consumer-facing software, embedded hardware, cloud backends, and third-party integrations, all in a context where the physical consequences of a security failure (someone unlocking your front door remotely) are very real.

The analysis covers system architecture, asset identification, STRIDE-based threat modeling, architectural security controls, and risk treatment. All diagrams were drawn in Mermaid and are in the `/Architecture` folder.


## Quick Summary

- **Threats identified:** 12 (across all STRIDE categories)
- **High-risk threats:** 5
- **Primary threat modeling framework:** STRIDE
- **Key controls:** mTLS device auth, VLAN segmentation, end-to-end encryption, immutable audit logs, signed OTA firmware updates
- **One accepted residual risk:** Physical device tampering with direct hardware access

