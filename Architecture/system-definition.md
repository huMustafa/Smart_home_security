# Task 1: System Definition and Architecture

## Overview

The system I'm modeling is a smart home IoT ecosystem,  the kind of setup where you can lock your front door, dim the lights, check a camera feed, and yell at a speaker to turn off the AC, all from your phone. It sounds simple but the actual attack surface is surprisingly wide.


## Application Components

### 1. Mobile App (User Frontend)
The primary interface for homeowners. Lets users view device status, issue commands (lock/unlock, arm/disarm), review camera footage, and configure automation rules. Communicates with the backend over HTTPS REST APIs. Available on iOS and Android.

### 2. Web Admin Portal
A separate interface for system administrators — think the company that sells the smart home system. Used to manage firmware updates, monitor fleet health, and handle customer escalations. This is a privileged interface and should not be reachable from the same network path as the user app.

### 3. REST API Backend
The brain of the system. Handles authentication, routes commands from the app to the IoT hub, stores and retrieves device state, and proxies third-party integrations. Runs as a set of microservices (auth service, device service, notification service, etc.). Exposed to the internet but sits behind an API gateway.

### 4. IoT Hub / Local Gateway
A physical device installed in the home (think: similar to a router but for smart devices). Acts as the local coordinator, devices talk to the hub, and the hub talks to the cloud backend. This keeps local commands low-latency even if the internet goes down. Runs a lightweight MQTT broker.

### 5. Device Fleet
The actual smart devices:
- **Smart locks** — high-consequence physical actuators
- **IP cameras** — sensitive data producers (video/audio streams)
- **Smart lights and switches** — lower consequence but still part of the attack surface
- **Smart thermostat** — behavioral data (when you're home, sleep patterns)

Each device communicates with the hub over the local network (WiFi or Zigbee/Z-Wave).

### 6. Cloud Storage
Stores camera footage (encrypted at rest), device event logs, user preferences, and automation rules. Also used for firmware image hosting. Accessible to the API backend but not directly to users.

### 7. Third-Party Integrations
- **Alexa / Google Home** — voice control, uses OAuth for delegated access
- **Email/SMS Notification Service** — alerts for events like motion detected, door opened
- **Firmware Update Server** — delivers signed OTA firmware updates to devices via hub


## Users and Roles

| Role | Description | Access Level |
|---|---|---|
| Homeowner | Primary user of the app | Control own devices, view footage, configure automations |
| Guest User | Temporary access (e.g., houseguest) | Limited — specific devices only, time-bounded |
| Admin (Company) | Manages the platform | Fleet-wide access, firmware deployment, user management |
| Third-Party Service | Alexa, Google Home | Delegated, OAuth-scoped access only |


## Data Types Handled

- **Credentials** — usernames, passwords, OAuth tokens, device certificates
- **Personal/behavioral data** — when the user is home, sleep patterns, movement inside the home
- **Video/audio streams** — camera footage, possibly from inside the home
- **Device commands** — lock/unlock instructions, alarm arm/disarm
- **Firmware images** — update packages pushed to devices
- **Audit logs** — who did what, when


## External Dependencies

- Third-party voice assistant APIs (Alexa, Google Home)
- SMTP/SMS gateway for notifications
- Firmware CDN / update server
- Public internet (the whole system is internet-facing by design)


## Trust Boundaries

Trust boundaries are places where you can't just assume the thing on the other side is trustworthy. Here's how I've drawn them for this system:

1. **Internet ↔ API Gateway** — anything coming from the internet is untrusted. The API gateway is the first checkpoint.
2. **API Backend ↔ Cloud Storage** — backend services are trusted, but cloud storage should validate requests anyway (not blindly accept from anything claiming to be "the backend").
3. **API Backend ↔ IoT Hub** — the hub lives in the home, physically accessible. It should be treated as semi-trusted at best.
4. **IoT Hub ↔ Device Fleet** — individual devices are the least trusted. A compromised bulb shouldn't be able to issue commands to a lock.
5. **Admin Portal ↔ API Backend** — admin traffic crosses a separate trust boundary; admin functions should be on a different auth path entirely.
6. **Third-Party Integrations ↔ API Backend** — Alexa/Google Home are external and only get scoped, limited access via OAuth.


