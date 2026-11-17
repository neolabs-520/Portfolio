# Cybersecurity & Systems Engineering Portfolio

## Profile

I am a hands-on systems and security engineer focused on building, hardening, and operating resilient Linux infrastructure. My work is self-directed and lab-based: I research, deploy, and troubleshoot enterprise-grade technologies under real constraints — segmented networks, centralized logging, zero-trust remote access, and full lifecycle ownership from first boot to disaster recovery.

Two things shape how I work:

- **A security-first mindset.** I received early offensive and defensive exposure as a technical understudy to a CEH-certified security professional, which informs how I design controls and test them from an adversary's perspective.
- **Disciplined persistence.** I built rigorous habits around log analysis, syntax validation, and command-line diagnostics — the detail work this field demands.

All hostnames, IP addresses, usernames, and tokens in these write-ups are redacted or generalized.

---

## Core competencies

| Area | What I've done |
| :--- | :--- |
| **Linux systems administration** | Ubuntu Server (headless), Arch Linux (manual CLI install), kernel and package management, systemd services |
| **SIEM & observability** | Wazuh + OpenSearch deployment, agent enrollment, alert triage, MITRE ATT&CK mapping, Prometheus + Grafana |
| **Network security** | Network segmentation, route metrics, Tailscale ZTNA/WireGuard, Nmap reconnaissance, attack-surface reduction |
| **Vulnerability management** | CVE triage, Ubuntu Pro/ESM patching, forced package upgrades, documented risk acceptance |
| **Virtualization** | VirtualBox, KVM/QEMU (libvirt), Proxmox VE, VM provisioning and recovery |
| **Scripting & tooling** | Bash, Python, Git/GitHub, Streamlit, QML/Qt Quick |

---

## Lab architecture

A deliberately segmented home lab built on repurposed hardware and virtual machines. Administrative access runs over a Tailscale mesh; the SIEM has no path from untrusted local devices.

```text
                    [ Out-of-band attack node ]
                    Raspberry Pi · Kali Linux
                              │
                              ▼ Nmap recon / synthetic traffic
        ┌─────────────────────┴─────────────────────┐
        │       Tailscale zero-trust overlay        │
        │  WireGuard mesh · admin laptop · phone    │
        └─────────────────────┬─────────────────────┘
                              │
                    [ Wazuh Manager ]
                    Ubuntu Server 24.04 LTS
                    Wazuh · OpenSearch · Dashboard
                              │
                    [ Living-room endpoint ]
                    Ubuntu 22.04 LTS · Wazuh agent

        [ Isolated guest / IoT zone ] — segmented,
        no route to the SIEM or the admin plane
```

Supporting services include a local Prometheus/Grafana monitoring stack, an Ollama + Open WebUI instance for local LLM work, and a zero-authentication Plex server for the air-gapped client LAN.

---

## Case studies

### 1. SIEM deployment and first-run validation

**Challenge:** Stand up centralized security monitoring for a multi-host lab from scratch, on headless Linux, as a first-time Wazuh deployment.

**Action:** Researched, configured, and deployed the Wazuh/OpenSearch stack using the unattended installer. Built post-deployment verification checks and diagnosed a hostname configuration drift that was blocking initial log ingestion.

**Result:** Deployment completed successfully on the first attempt, with endpoint log ingestion verified end to end.

*Skills: SIEM architecture · Linux CLI · service management · validation testing*

---

### 2. Adversary simulation and detection validation (MITRE ATT&CK)

**Challenge:** Prove that the SIEM was actually parsing network anomalies and generating accurate alerts — not just collecting logs.

**Action:** Provisioned an out-of-band Raspberry Pi running Kali Linux as a dedicated threat-actor node. Ran Nmap SYN stealth scans and OS fingerprinting against core infrastructure to generate synthetic malicious traffic, then traced the resulting telemetry through the pipeline.

**Result:** Alerts fired in real time. Endpoint agents captured the scan footprint, forwarded logs to the manager, and the events mapped to **MITRE ATT&CK TA0043 (Reconnaissance)** and **T1595 (Active Scanning)** in the dashboard.

*Skills: Nmap · attack simulation · detection engineering · ATT&CK mapping*

---

### 3. Zero-trust remote administration (Tailscale)

**Challenge:** Administer the lab remotely without exposing SSH — or any other management port — to the public internet.

**Action:** Overlaid a Tailscale network across all administrative assets and moved access to Tailscale SSH, with node-level authorization and end-to-end encrypted WireGuard transport.

**Result:** WAN-facing SSH was removed entirely. Administrative access now requires a pre-authenticated mesh node, and works behind CGNAT from any external network.

*Skills: ZTNA · WireGuard · access control · attack-surface reduction*

---

### 4. Network segmentation and perimeter isolation

**Challenge:** The lab shares physical space and an upstream network with household IoT and guest devices that must never reach the SIEM or admin plane.

**Action:** Segmented untrusted client traffic into an isolated zone. The client LAN runs on a separate WLAN interface with a high route metric and no internet path, while the management plane stays reachable only over Tailscale. As a compliance exercise, I mapped the design to **HIPAA** and **DORA** principles and prioritized log sanitization before external debugging.

**Result:** No path from guest/IoT devices to the SIEM. Lateral-movement surface is reduced by construction rather than by a software firewall (UFW is intentionally disabled).

*Skills: network segmentation · routing · risk assessment · compliance mapping*

---

### 5. SIEM database recovery after ungraceful shutdown

**Challenge:** A hypervisor abort corrupted the OpenSearch index shards. The dashboard returned Axios authorization-token errors, and the indexer rejected cluster registration on port 9200.

**Action:** Performed root-cause analysis, cleared stale lock files (`/var/run/wazuh-indexer.pid`, `/var/ossec/var/run/*.pid`), corrected a persistent hostname typo in `/etc/hosts`, and brought services up in a strict sequence: indexer, then manager, then dashboard.

**Result:** Cluster communication and log ingestion restored with no data loss.

*Skills: RCA · OpenSearch · systemd · disaster recovery*

---

### 6. Vulnerability lifecycle management (patching vs. risk acceptance)

**Challenge:** Endpoint A tracked **327 vulnerabilities**, including **CVE-2023-51714** (CVSS 9.8, a Qt HTTP/2 HPACK integer-overflow vulnerability) that standard `apt` upgrades could not resolve because the affected packages were held back.

**Action:** Enrolled the host in Ubuntu Pro (ESM Apps) to pull backported fixes, then ran a targeted upgrade of the held packages and re-scanned. Separately evaluated **CVE-2021-3773** (netfilter/OpenVPN information exposure) and documented it as a formal accepted risk, justified by the endpoint's strict isolation from untrusted networks.

**Result:** The critical attack vector was remediated, and the tracked vulnerability count dropped from **327 to 288**. The risk acceptance was recorded with its rationale.

*Skills: CVE triage · Ubuntu Pro/ESM · dependency management · risk documentation*

---

### 7. Low-level Linux kernel and dependency recovery

**Challenge:** Endpoint B's package manager was locked (`dpkg error code 1`) after the third-party `rtl8852bu` Wi-Fi driver failed to compile against kernel `6.8.0-147-generic`.

**Action:** Force-removed the broken kernel header packages to release the compilation lock, repaired dependencies with `apt-get install -f`, and reattached the host to the shared Ubuntu Pro subscription.

**Result:** Package database repaired, stale telemetry cleared, and the host returned to a managed, patchable state.

*Skills: dpkg · kernel modules · dependency resolution · system recovery*

---

## Detailed write-ups

- [Wazuh SIEM troubleshooting case study](case-studies/wazuh-siem-troubleshooting.md) · [PDF](case-studies/wazuh-siem-troubleshooting.pdf)
- [Home lab architecture and design notes](labs/homelab-architecture.md)
- [Wazuh SIEM deployment on headless Ubuntu](labs/wazuh-siem-deployment.md)
- [Arch Linux manual installation and desktop recovery](labs/arch-linux-installation.md)
- [Arch Linux installation blueprint](labs/arch-linux-install-blueprint.md)
- [Arch Linux systems administration lab guide](labs/arch-linux-sysadmin-lab.md)

---

## Supporting software projects

**NeoLabs landing site** — Static website built and maintained with Git/GitHub. Demonstrates web fundamentals and a publishing workflow.  
`https://github.com/neolabs-520/NeoLabs`

**DippEdc** — An Omarchy bar widget / everyday-carry checklist built with QML and JavaScript. Includes persistent on-disk state, keyboard-driven workflows, localization (`i18n`), a completion celebration overlay, and Node-based model tests. Demonstrates end-to-end desktop feature work with a clean UI/model separation.

**The Rotten Meme Store** — A modular Python/Streamlit image-editing app with separate layout, theme, filter, and watermarking modules. Demonstrates application structure, asset/template handling, and UI-orchestration separation.

---

## Certifications & training

- **Cisco Networking Academy — Cybersecurity Basics** (badge earned)  
  [Verify credential](https://www.credly.com/badges/ee5691bc-6ab4-44e9-900c-446141280158/public_url)
- **freeCodeCamp — HTML course** (in progress), applied through personal web projects

---

## Ways of working

- **Work in small, verifiable increments.** I scope tasks so each change is testable, summarize state before a working set gets unwieldy, and keep a decision log. In long AI-assisted sessions, that means compacting context before the window saturates.
- **Stay unblocked.** When a tool, provider, or service fails, I switch to an equivalent instead of waiting, and keep a local fallback for offline work. No single dependency should stall the job — the same principle as redundant paths in the lab.
- **Verify, then execute.** Commands — whether I write them, copy them, or get them from an AI assistant — are read, sanity-checked, and tested in a disposable environment before they touch a real host.
- **Document the recovery path.** Every dependency gets a fallback and a written procedure, so a single failure is an inconvenience rather than a crisis. Rebuildability is a security property.

---

## Current focus

- Publishing redacted case studies and operational runbooks
- Expanding automated tests and CI/CD across software projects
- Seeking roles in SOC analysis, systems administration, or security engineering

---

## Contact

- **GitHub:** https://github.com/neolabs-520
- **LinkedIn:** *(to add)*
- **Website / portfolio:** *(in progress)*
- **Email:** available on request
