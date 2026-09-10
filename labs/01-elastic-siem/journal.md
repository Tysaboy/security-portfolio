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
