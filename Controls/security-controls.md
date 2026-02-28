# Task 4: Secure Architecture Design

These are architectural controls — I'm not suggesting code changes, just describing how the system should be structured to reduce the threats identified in Task 3.


## Security Controls

### 1. Identity and Access Management

**Control:** Implement mutual TLS (mTLS) for device-to-hub communication. Each device gets a unique client certificate provisioned at manufacturing time. The hub only accepts connections from devices presenting a valid, unexpired certificate signed by the platform's CA.

**Justification:** Directly addresses T1 (device spoofing). MAC address-based auth is trivially bypassable. mTLS makes spoofing a specific device require compromising its private key, which is much harder if keys are stored in a device-side secure element. [web:21]

**Control:** OAuth 2.0 with PKCE for mobile app authentication. Short-lived access tokens (15-minute expiry), refresh tokens stored in secure enclave on the device.

**Justification:** Limits the damage of token theft. A stolen token that expires in 15 minutes is much less useful than one that lasts days.

**Control:** Admin portal access restricted to authenticated admin users with hardware MFA (TOTP minimum, physical security key preferred). Admin accounts separate from user accounts — no admin/user account overlap.

**Justification:** Addresses T9. If you have MFA and admin accounts are separate from the user database, a compromised user account can't escalate to admin.



### 2. Network Segmentation

**Control:** IoT devices and the hub are on a dedicated VLAN, isolated from the main home network and from each other where possible. The hub is the only device on the IoT VLAN allowed to initiate outbound connections to the API backend.

**Justification:** Directly addresses T8. If a smart bulb is compromised, it can't reach the lock or the hub's admin interface because they're on separate network segments. The blast radius of any single device compromise is contained. [web:23]

**Control:** Admin portal is not exposed on the public internet. Accessible only via VPN, with the VPN gateway behind the API layer.

**Justification:** Addresses T9. Removing the admin portal from the internet-facing attack surface eliminates the easiest path to platform-wide compromise.

**Control:** Separate API gateway paths for user traffic and admin traffic. Different rate limiting, logging verbosity, and auth requirements per path.

**Justification:** Defense-in-depth. Even if the user-facing API is compromised, the admin path has additional checkpoints.


### 3. Data Protection

**Control:** TLS 1.3 for all data in transit — app ↔ API, API ↔ hub, hub ↔ cloud storage. No TLS 1.1 or 1.2 allowed (deprecated cipher suites disabled).

**Justification:** Addresses T2 and T4. Encrypting data in transit is the baseline, but specifying TLS 1.3 removes a class of known vulnerabilities in older TLS versions.

**Control:** Camera footage encrypted at rest using AES-256. Encryption keys managed separately from the data (not co-located in the same storage bucket).

**Justification:** Addresses T4. If someone exfiltrates the storage bucket, they get ciphertext, not usable footage.

**Control:** Event logs and behavioral data access requires row-level authorization checks — users can only retrieve their own data. No predictable numeric IDs for user or device records (use UUIDs).

**Justification:** Addresses T5. Prevents IDOR attacks where an attacker increments a user ID to access another user's data.


### 4. Secrets Management

**Control:** API keys, database credentials, and service-to-service tokens stored in a secrets manager (HashiCorp Vault or equivalent, not environment variables or config files). Secrets rotated automatically on a schedule.

**Justification:** Addresses A6. Hardcoded secrets in config files are the easiest credentials to steal — they get committed to git, left in deployment artifacts, etc. A dedicated secrets manager with audit logging is just better practice. [web:19]

**Control:** OAuth scopes for third-party integrations (Alexa, Google Home) are minimal and device-type specific. Voice assistants get write access to lights and thermostat only — not locks or cameras.

**Justification:** Addresses T11. Least-privilege OAuth scopes mean a compromised Alexa integration can't unlock your front door.


### 5. Monitoring and Logging

**Control:** Immutable audit logs for all security-relevant actions (login, device command, firmware update, permission change). Logs shipped to a write-once, append-only log store. Logs include user ID, timestamp, source IP, session token hash, and action.

**Justification:** Addresses T3 and T12. Immutable logs can't be modified post-hoc. Detailed enough logs make repudiation actually hard.

**Control:** Anomaly detection on device behavior — baseline normal patterns, alert on deviation (a lock that was never triggered overnight suddenly being commanded at 3am from a new IP is worth flagging).

**Justification:** Helps catch T1, T8. Behavioral baselines don't prevent attacks but reduce the time-to-detection significantly.

**Control:** Admin actions logged separately with enhanced verbosity and forwarded to a SIEM. Any admin login from a new IP triggers immediate alert.

**Justification:** Admin account compromise (T9) should be detected immediately, not days later.


### 6. Secure Deployment Practices

**Control:** All firmware updates signed with a private key held offline. Hub verifies signature before installing. Updates delivered over TLS. No unsigned firmware accepted under any circumstances.

**Justification:** Addresses T10. Signed firmware with offline keys means even if the update server is compromised, attackers can't push malicious firmware without the signing key.

**Control:** Devices ship with unique default credentials (not a shared factory default). First-time setup forces credential change.

**Justification:** Addresses T1 and T8 at the device level. Shared default credentials are one of the most common IoT attack vectors.

**Control:** Unused ports and services disabled on all devices and the hub at the factory. Periodic automated scans to detect configuration drift.

**Justification:** Reduces the attack surface of each device. If a port isn't open, it can't be exploited.


