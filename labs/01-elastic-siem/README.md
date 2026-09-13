# Lab 01 — Elastic SIEM + Kali Attack Lab

## Problem Statement

Can a home lab SIEM reliably detect an SSH brute force attack in real time, 
from attacker to alert, with zero false positives on baseline traffic?

## Threat Context

**MITRE ATT&CK:** T1110 — Brute Force (Credential Access, TA0006)

SSH brute force is one of the most common initial access techniques against 
internet-exposed Linux systems. In 2024, the Mirai botnet variants used this 
exact technique to compromise hundreds of thousands of IoT devices by 
credential stuffing default SSH passwords.

## Architecture

| Device | Role | IP |
|--------|------|----|
| MacBook UTM — Ubuntu 24.04 | SIEM host (ELK in Docker) | 192.168.0.218 |
| MacBook UTM — Kali Linux | Attack target | 192.168.0.77 |
| Raspberry Pi 5 | Attacker | 192.168.0.18 |

## Environment

Reproduced with:

```bash
git clone https://github.com/Tysaboy/security-portfolio
cd labs/01-elastic-siem
docker compose up -d
```

Requires Docker and Docker Compose on an ARM64 or x86 Linux host.

## Attack Execution

**Attack 1 — Port reconnaissance:**
```bash
sudo nmap -sS -sV 192.168.0.77
```

**Attack 2 — SSH credential brute force:**
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt \
  ssh://192.168.0.77 -t 4 -V
```

Result: 4,324 failed authentication attempts generated over 20 minutes.
Source IP 192.168.0.18 made 100% of the attempts.

## Detection Artifact

**KQL query (Kibana Discover):**
event.dataset: "system.auth" and
host.name: "kali" and
system.auth.ssh.event: "Failed"


**Threshold detection rule:**
Index: logs-system.auth-*
Query: event.action: "ssh_login" and event.outcome: "failure"
Group by: source.ip
Threshold: >= 10 in 1 minute
Severity: High
Risk score: 73
MITRE: T1110 — Brute Force


## Validation

**True positive:** Rule fired 37 seconds after enabling, detecting 
36 failed logins from 192.168.0.18 within a single 1-minute window.

**False positive check:** Ran the same rule against 24 hours of 
baseline traffic (no attacks). Zero alerts generated. Rule stayed 
quiet on normal sudo and SSH key-based logins.

## What Broke

1. Fleet Server crashed on first start because it tried to fetch a 
   service token from Kibana before Kibana finished initializing. 
   Fixed by manually generating a service token via Elasticsearch API.

2. Kibana UI rejected HTTP Fleet Server URLs. Bypassed by registering 
   the host via the Kibana REST API directly.

3. Fleet Server stuck in DEGRADED state because the default Fleet 
   Server policy didn't exist after docker compose down wiped saved 
   objects. Fixed by creating the policy via API and passing 
   FLEET_SERVER_POLICY_ID explicitly.

4. auth.log missing on Kali 2026.1 — modern Kali uses journald only. 
   Fixed by installing rsyslog to bridge journald to traditional log 
   files that Elastic Agent's System integration expects.

## Defensive Recommendation

Enable threshold-based SSH brute force detection on all Linux hosts 
exposing port 22. Accept the tradeoff: a threshold of 10 failures 
per minute will miss slow-and-low attacks (1 attempt per minute over 
hours) but eliminates alert fatigue from noisy automated scanners.

For higher sensitivity: combine with geolocation enrichment to flag 
logins from unexpected countries, accepting ~5% false positive rate 
from VPN users.

Immediate hardening: disable password authentication entirely 
(`PasswordAuthentication no` in sshd_config) and require SSH keys. 
This makes brute force impossible regardless of detection coverage.

## Resume Bullets

- Deployed Elastic SIEM (ELK 8.13) in Docker on ARM64; ingested 
  Linux auth logs via Elastic Agent Fleet enrollment
- Simulated SSH brute force from Raspberry Pi 5; generated 4,324 
  authentication failure events detected in real time
- Authored threshold detection rule in Kibana mapped to MITRE T1110; 
  validated true positive rate with zero false positives on baseline
- Built SOC monitoring dashboard tracking attack timeline, source IPs, 
  and login outcome breakdown
