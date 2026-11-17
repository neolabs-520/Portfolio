# Cybersecurity & Systems Engineering Portfolio

Hands-on systems and security work: building, hardening, and operating Linux infrastructure in a deliberately segmented home lab.

**[View the live portfolio →](https://neolabs-520.github.io/Portfolio/)**

I focus on the parts of the job that are hard to fake — deploying a SIEM from scratch, proving it detects an actual adversary, recovering a corrupted cluster, and making an endpoint patchable again when the package manager is broken.

---

## Start here

**[Cybersecurity & Systems Engineering Portfolio](cybersecurity-portfolio.md)** — profile, core competencies, lab architecture, and seven detailed case studies.

**[Wazuh SIEM Troubleshooting Case Study](case-studies/wazuh-siem-troubleshooting.md)** ([PDF](case-studies/wazuh-siem-troubleshooting.pdf)) — a redacted walkthrough of vulnerability remediation and disaster recovery across a multi-host lab.

**[Home Lab Architecture & Design Notes](labs/homelab-architecture.md)** — the shareable design summary: trust zones, storage tiering, virtualization, observability, and the trade-offs behind each decision.

---

## Featured work

| Project | What it demonstrates |
| :--- | :--- |
| **Wazuh SIEM deployment & detection validation** | Deployed Wazuh/OpenSearch on headless Ubuntu. Built an out-of-band Kali node and ran Nmap scans to validate detection, mapped to MITRE ATT&CK TA0043 / T1595. |
| **Vulnerability remediation & SIEM recovery** | Removed CVE-2023-51714 (CVSS 9.8) via Ubuntu Pro/ESM, reduced tracked vulnerabilities from 327 to 288, and recovered a corrupted OpenSearch cluster with no data loss. |
| **Zero-trust remote administration** | Replaced WAN-facing SSH with Tailscale SSH over an encrypted WireGuard mesh, with node-level authorization. |
| **Network segmentation** | Isolated guest/IoT traffic from the SIEM and admin plane using separate WLAN interfaces, route metrics, and a zero-trust overlay. |
| **Arch Linux manual install** | Installed Arch from the CLI, recovered a 100% full root filesystem, and stabilized GNOME by forcing X11 over a problematic Wayland/VMSVGA combination. |

Supporting software projects: **NeoLabs** (static site, Git/GitHub), **DippEdc** (QML/JavaScript desktop widget with tests and persistence), and **The Rotten Meme Store** (modular Python/Streamlit app).

---

## Repository map

```text
.
├── cybersecurity-portfolio.md          # main portfolio document
├── case-studies/
│   ├── wazuh-siem-troubleshooting.md   # redacted incident case study
│   └── wazuh-siem-troubleshooting.pdf
└── labs/
    ├── homelab-architecture.md          # sanitized design notes
    ├── wazuh-siem-deployment.md         # first-run SIEM deployment
    ├── arch-linux-installation.md       # manual install + recovery
    ├── arch-linux-install-blueprint.md  # repeatable install procedure
    └── arch-linux-sysadmin-lab.md       # full lab manual
```

---

## How I work

- **Work in small, verifiable increments.** I scope tasks so each change is testable, summarize state before a working set gets unwieldy, and keep a decision log. In long AI-assisted sessions, that means compacting context before the window saturates.
- **Stay unblocked.** When a tool, provider, or service fails, I switch to an equivalent instead of waiting, and keep a local fallback for offline work.
- **Verify, then execute.** Commands — written, copied, or AI-suggested — are read, sanity-checked, and tested in a disposable environment before they touch a real host.
- **Document the recovery path.** Every dependency gets a fallback and a written procedure.

---

## Certifications & training

- **Cisco Networking Academy — Cybersecurity Basics** ([verify badge](https://www.credly.com/badges/ee5691bc-6ab4-44e9-900c-446141280158/public_url))
- **freeCodeCamp — HTML course** (in progress)

---

## Contact

- **GitHub:** [neolabs-520](https://github.com/neolabs-520) — open an issue or reach out there
- **LinkedIn:** *(to add)*

All hostnames, IP addresses, usernames, and tokens in these write-ups are redacted or generalized.
