**Wazuh SIEM + Cowrie Honeypot Lab
Overview**

This project is a home-built SIEM and honeypot lab designed to gain hands-on detection engineering and incident analysis experience for security analyst roles. It combines an open-source SIEM (Wazuh) monitoring a real endpoint with detailed telemetry (Sysmon), alongside a separate, internet-facing SSH honeypot (Cowrie) used to capture and analyze real attacker behavior rather than simulated data. The goal was to move beyond theoretical coursework and build practical, defensible experience with the same tools and workflows used in real SOC/detection engineering environments — deployment, configuration, troubleshooting, and analysis of genuine attack traffic.

Key Findings
Identified and remediated a default-credential vulnerability (admin/admin) on the Wazuh management dashboard before exposing any part of the stack to real traffic
Wazuh + Sysmon integration mapped endpoint activity to 616 Privilege Escalation and 605 Defense Evasion events (MITRE ATT&CK) on a single monitored endpoint in 24 hours
CIS Microsoft Windows 11 Enterprise Benchmark v3.0.0 scan returned a 25% compliance score (122 passed / 350 failed / 10 not applicable) on a default Windows 11 install — a concrete, quantified hardening gap
Cowrie honeypot, exposed on a public DigitalOcean droplet, captured real unsolicited attacker sessions within hours of deployment, including:
Credential-stuffing attempts
An SSH key persistence (backdoor) attempt
Evidence of anti-honeypot reconnaissance tooling
Full findings, IOCs, and remediation recommendations documented in [incident-analysis.md](url)

Tech Stack
SIEM: Wazuh (manager + dashboard)
Endpoint telemetry: Sysmon (Windows)
Honeypot: Cowrie (SSH)
Virtualization: VirtualBox (Wazuh manager VM)
Cloud hosting: DigitalOcean (honeypot droplet)
Network/traffic control: iptables (port redirection for honeypot)
OS monitored: Windows 11 Home

**Architecture**
Wazuh manager + dashboard — deployed as a VirtualBox VM on a local network, providing centralized log collection, alerting, and a web dashboard for analysis.
Monitored endpoint — a personal Windows laptop running the Wazuh agent, enhanced with Sysmon for high-fidelity process, network, and system-level telemetry beyond Windows' default event logging.
Cowrie SSH honeypot — deployed on a public-facing DigitalOcean cloud VM (droplet), intentionally exposed to the open internet to attract and capture real, unsolicited attacker traffic rather than staged activity.
[ Personal Laptop ] --Sysmon + Wazuh Agent--> [ Wazuh Manager/Dashboard (VirtualBox VM) ]

[ Internet ] --> [ Cowrie Honeypot (DigitalOcean Droplet) ] --> cowrie.json logs

**What I Built**
Deployed and secured a Wazuh SIEM, including identifying and remediating a default-credential vulnerability (admin/admin) on the management dashboard using the platform's official password-reset tooling.
Connected a real endpoint to the SIEM and integrated Sysmon to capture process creation, network connections, and other high-fidelity telemetry that Windows' default logging does not provide.
Deployed a public-facing Cowrie SSH honeypot on a cloud VM, including moving real administrative SSH access to a non-standard port and configuring iptables traffic redirection so the honeypot could safely impersonate a standard SSH service.
Captured and analyzed real, unsolicited attacker sessions within hours of deployment, identifying credential-stuffing patterns, an SSH key persistence (backdoor) attempt, and evidence of anti-honeypot reconnaissance tooling.
Documented findings in a formal incident analysis, including indicators of compromise and remediation recommendations.

Setup / Reproduction

High-level steps to reproduce this lab. Substitute your own IPs, credentials, and hostnames.

1. Deploy the Wazuh manager

Spin up a VirtualBox VM (Ubuntu recommended) on your local network
Install Wazuh manager + dashboard following the official Wazuh install guide
Log in to the dashboard and immediately reset the default admin password using Wazuh's built-in password-reset tooling

2. Connect an endpoint

Install the Wazuh agent on the machine you want to monitor (Windows in this case)
Register the agent to the manager and confirm it shows active on the Endpoints page
Install Sysmon with a hardening-focused config (e.g. SwiftOnSecurity's or Olaf Hartong's) to get high-fidelity process, network, and registry telemetry beyond Windows' default event logging
Point Wazuh's Windows agent config at the Sysmon event channel so those events are ingested

3. Deploy the honeypot

Provision a DigitalOcean droplet (cheapest tier is sufficient)
Install Cowrie SSH honeypot
Move real administrative SSH access off port 22 to a non-standard port
Configure iptables to redirect inbound traffic on port 22 to Cowrie, so it convincingly impersonates a standard SSH service to attackers while your real admin access stays isolated
Point Cowrie's JSON log output somewhere you can review it (locally, or ship to Wazuh/a log pipeline)

4. Validate

Confirm the Wazuh dashboard shows the endpoint as active and events flowing in (Overview → Agents Summary)
Confirm Cowrie is logging connection attempts (cowrie.json)
Run a Security Configuration Assessment scan against the endpoint to get a CIS benchmark baseline

5. Analyze

Use Wazuh's Threat Hunting / MITRE ATT&CK views to triage and tag alerts by tactic
Review Cowrie logs for attacker sessions, credential attempts, and any commands executed
Document findings, IOCs, and remediation steps (see [incident-analysis.md](url))

## Screenshots
<img width="2554" height="1394" alt="image" src="https://github.com/user-attachments/assets/d17495f6-09d6-4541-9688-d42ba6756e07" />

<img width="2559" height="1328" alt="image" src="https://github.com/user-attachments/assets/6b3c56d0-ab6f-42f9-82f7-38570c31a052" />

