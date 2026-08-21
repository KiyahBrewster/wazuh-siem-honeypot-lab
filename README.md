**Wazuh SIEM + Cowrie Honeypot Lab
Overview**

This project is a home-built SIEM and honeypot lab designed to gain hands-on detection engineering and incident analysis experience for security analyst roles. It combines an open-source SIEM (Wazuh) monitoring a real endpoint with detailed telemetry (Sysmon), alongside a separate, internet-facing SSH honeypot (Cowrie) used to capture and analyze real attacker behavior rather than simulated data. The goal was to move beyond theoretical coursework and build practical, defensible experience with the same tools and workflows used in real SOC/detection engineering environments — deployment, configuration, troubleshooting, and analysis of genuine attack traffic.

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

## Screenshots
<img width="2554" height="1394" alt="image" src="https://github.com/user-attachments/assets/d17495f6-09d6-4541-9688-d42ba6756e07" />

<img width="2559" height="1328" alt="image" src="https://github.com/user-attachments/assets/6b3c56d0-ab6f-42f9-82f7-38570c31a052" />

