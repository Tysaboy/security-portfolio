# Lab 01 — Elastic SIEM + Kali Attack Lab

## Journal

---

### September 9, 2026

**What was accomplished:**
- Rebuilt the Elastic SIEM lab from scratch on a cleaner architecture (MacBook UTM instead of Lenovo)
- Confirmed Ubuntu 24.04 LTS ARM64 as the SIEM host — native Apple Silicon, no emulation
- Installed Docker 29.8.0 and Docker Compose v5.5.1 on Ubuntu UTM VM
- Deployed ELK stack (Elasticsearch 8.13.0 + Kibana 8.13.0 + Fleet Server 8.13.0) via Docker Compose
- Resolved Fleet Server authentication failure by manually generating a service token via Elasticsearch API instead of relying on Kibana auto-provisioning
- Set `restart: always` on all containers so the stack survives reboots
- Confirmed Kibana accessible at `http://192.168.0.218:5601`
- Configured Kali UTM VM as attack target: SSH, FTP (vsftpd), Apache2 installed and running
- Disabled SSH PerSourcePenalties on target to allow unrestricted brute force simulation
- Confirmed network connectivity between Ubuntu SIEM (192.168.0.218) and Kali target (192.168.0.77)

**Blocked on:**
Fleet Server HTTPS requirement — Kibana rejects HTTP Fleet Server URLs in the UI. Fix scheduled for next session.

**What broke and why:**
- Fleet Server crashed on first start because it tried to request a service token from Kibana before Kibana finished initializing. Fixed by manually generating the token via `curl` against Elasticsearch directly.
- `elastic` user password mismatch after container restart. Fixed using `elasticsearch-reset-password -u elastic -i` inside the running container.

**Next session:**
- Fix Fleet Server HTTPS issue
- Register Fleet Server host in Kibana Settings
- Update Elasticsearch output from `localhost:9200` to `192.168.0.218:9200`
- Enroll Elastic Agent on Kali target VM
- Confirm logs flowing in Kibana Discover

**Architecture:**
- SIEM host: Ubuntu 24.04 UTM VM — 192.168.0.218
- Target: Kali Linux UTM VM — 192.168.0.77
- Attacker: Raspberry Pi 5 — 192.168.0.18


### September 10-11, 2026

**What was accomplished:**
- Fixed Fleet Server HTTPS issue by switching to FLEET_SERVER_INSECURE_HTTP=true
- Discovered correct elastic-agent container environment variable names from --help output
- Registered Fleet Server host in Kibana via API (bypassing UI HTTP validation)
- Created missing Fleet Server policy via Kibana API
- Added FLEET_SERVER_POLICY_ID=fleet-server-policy to docker-compose.yml
- Fleet Server now running HEALTHY and visible in Kibana Fleet → Agents
- Kibana Elasticsearch output updated to http://192.168.0.218:9200

**What broke and why:**
- Fleet Server kept failing with "url is required when a certificate is provided" — attempted SSL cert approach was unnecessary complexity for a home lab
- Fleet Server was stuck in DEGRADED state because the default Fleet Server policy didn't exist in Kibana after docker compose down wiped saved objects
- Fixed by creating the policy via API and explicitly passing FLEET_SERVER_POLICY_ID

**Key concepts learned:**
- elastic-agent container uses specific environment variable names, not CLI flags
- docker compose down removes saved Kibana objects — always use docker compose restart instead to preserve state
- Fleet Server needs three things to reach HEALTHY: service token, policy ID, and Kibana registered host URL

**Next session:**
- Confirm Kali VM is ARM64 architecture
- Install rsyslog on Kali target VM
- Download elastic-agent ARM64 build
- Enroll Elastic Agent on Kali target using Kali-Target policy enrollment token
- Confirm logs flowing in Kibana Discover

**Architecture:**
- SIEM host: Ubuntu 24.04 UTM VM — 192.168.0.218
- Target: Kali Linux UTM VM — 192.168.0.77
- Attacker: Raspberry Pi 5 — 192.168.0.18



### September 12, 2026

**What was accomplished:**
- Elastic Agent successfully enrolled on Kali target VM (ARM64)
- Confirmed logs flowing in Kibana Discover — 838 system.auth documents
- SSH brute force attack detected: 4,324 failed login attempts from 192.168.0.18
- Detection rule fired: SSH Brute Force - Pi Attack Lab (High, risk score 73, T1110)
- Built SOC dashboard with 3 panels: failed logins timeline, top attacking IPs, outcome breakdown
- Lab fully complete end to end

**Lab status: COMPLETE**
