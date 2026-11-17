# Wazuh SIEM Deployment: Headless Ubuntu Server

**Environment:** Ubuntu Server VM (headless)  
**Objective:** Deploy an all-in-one Wazuh SIEM instance to provide centralized security monitoring  
**Date:** October 2026

---

## Overview

This project documents the deployment of the **Wazuh open-source security platform** on a headless Ubuntu Server VM. It covers the unattended installation, network verification, an administrative credential recovery, and first access to the web dashboard.

---

## Step 1 — Automated deployment

Installed the stack using the official unattended assistant, which handles dependency validation, internal PKI certificates, indexer setup, and web service configuration:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**Components deployed:**

- **Wazuh Indexer** — cluster node parameters and certificate structure configured
- **Wazuh Server** — Filebeat and internal communication channels verified
- **Wazuh Dashboard** — server-side components and browser bundle compiled (approximately 15 minutes)

---

## Step 2 — Network identification and topology verification

Because the server runs headless, the active interface and address were verified from the CLI before attempting remote access:

```bash
ip a
```

**Observed:**

- `lo` — loopback at `127.0.0.1/8`
- `enp0s3` — active Ethernet interface (`UP`, `LOWER_UP`)
- Host IPv4 address — assigned dynamically as `<LAN-IP>/24` with broadcast via `<broadcast-IP>`

---

## Step 3 — Administrative credential recovery

The unattended installer printed a generated administrator password to standard output. The string contained visually ambiguous characters — for example, lowercase `o` vs. uppercase `O`, `0` vs. `O`, and uppercase `Q` — and the initial login failed with "Invalid username or password."

Rather than guessing, the password was reset directly through the indexer's backend tool:

```bash
# Reset the admin account through the OpenSearch security tool
sudo bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh \
  -u admin -p '<new-password>'

# Restart the dashboard to apply the change
sudo systemctl restart wazuh-dashboard
```

> Operational note: the installer-generated password is intentionally not reproduced in this write-up. The reset credential is handled out of band and was not committed to documentation.

---

## Step 4 — Web access and validation

With the network and credentials in place, the dashboard was reached over HTTPS from another host on the LAN:

- **URL:** `https://<LAN-IP>`
- **Certificate:** self-signed; browser warning reviewed and accepted for the lab environment
- **Result:** authenticated access to the Wazuh Security Dashboard, with the manager, indexer, and dashboard all reporting healthy

---

## Result

The all-in-one Wazuh deployment completed successfully on the first attempt. Centralized log collection and the web management dashboard were operational, providing the foundation for the detection and vulnerability work described in the portfolio case studies.

---

## Skills demonstrated

- **Linux server administration** — headless CLI, network interface inspection, `systemctl` service management
- **SIEM deployment** — Wazuh manager, indexer, and dashboard components; certificate and dependency handling
- **Identity and access management** — backend credential reset, password hygiene, post-change service restart
- **Network validation** — interface state, addressing, and cross-host HTTPS access
