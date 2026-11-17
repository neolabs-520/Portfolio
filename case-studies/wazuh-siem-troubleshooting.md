# Wazuh SIEM Troubleshooting Case Study (Redacted)

**Scope:** Vulnerability remediation and SIEM cluster recovery across a multi-host lab  
**Environment:** Ubuntu 22.04 / 24.04 LTS endpoints, Wazuh 4.x (manager, indexer, dashboard), Ubuntu Pro / ESM, VirtualBox / Proxmox VE  
**Date:** October 2026

> This is a redacted version prepared for portfolio sharing. Hostnames, usernames, IP addresses, and tokens have been generalized.

---

## 1. Executive summary

This case study covers three linked incidents in a production-style test network:

1. A **critical CVSS 9.8 vulnerability** with a potential remote-code-execution vector on an Ubuntu endpoint that standard `apt` upgrades could not fix.
2. A **broken package-manager state** on a second endpoint, caused by a third-party Wi-Fi driver failing to compile against an updated kernel.
3. A **corrupted Wazuh/OpenSearch cluster** after an ungraceful hypervisor shutdown.

All three were resolved. The critical CVE was eliminated, the tracked vulnerability count fell from **327 to 288**, the package database was repaired, and the SIEM cluster was restored **without data loss**.

---

## 2. Phase I — Threat triage and endpoint hardening

### Finding

Exposure analysis in the Wazuh dashboard identified **CVE-2023-51714** (CVSS 9.8) on an Ubuntu 22.04.5 LTS host. The vulnerability is an incorrect HPACK integer-overflow check in Qt's HTTP/2 implementation, network-reachable and unauthenticated, with a potential remote-code-execution impact. The host tracked **327 vulnerabilities** in total.

### Problem

Standard `apt` upgrades did not resolve the CVE because the affected packages were held back.

### Action

Registered the endpoint with an active **Ubuntu Pro (ESM Apps)** subscription to pull backported fixes, then bypassed the held packages with a targeted upgrade:

```bash
sudo apt-get --with-new-pkgs upgrade jellyfin jellyfin-server -y
```

### Result

CVE-2023-51714 was removed from the active threat inventory and the endpoint's tracked vulnerability count dropped from **327 to 288**.

---

## 3. Phase II — Broken package state and ESM onboarding

### Finding

Replicating the hardening work on a second endpoint (`<endpoint-host>`) failed: the system-wide installer was locked and `dpkg` returned **error code 1**. The root cause was a failed compilation of the third-party `rtl8852bu` Wi-Fi driver against kernel `6.8.0-147-generic`, which left corrupted kernel headers behind.

> Note: a separate Axios authorization-token error was appearing in the Wazuh dashboard at the same time. It was traced to the cluster outage in Phase III, not to this endpoint.

### Action

Removed the broken headers to release the package-manager lock, repaired dependencies, and reattached the host to the shared subscription:

```bash
sudo dpkg --remove --force-remove-reinstreq linux-headers-generic linux-headers-6.8.0-147-generic
sudo apt-get install -f
sudo pro detach && sudo pro attach <TOKEN>
```

### Result

The package database was repaired, the host was unified under the enterprise security subscription, and patching resumed normally.

---

## 4. Phase III — Distributed SIEM crash and recovery

### Finding

After an ungraceful hypervisor abort of the Wazuh Manager VM, the dashboard lost connectivity and entered repeated `AxiosError: Error getting the authorization token` loops. On restart, the server refused to come up cleanly:

- Abandoned PID files were blocking service startup.
- OpenSearch index shards were in a corrupt state and rejected cluster registration on port 9200.
- A persistent hostname typo in `/etc/hosts` was preventing the manager from resolving the indexer.

### Action

Corrected the hostname, cleared stale application locks, and brought the stack up in strict dependency order:

```bash
sudo hostnamectl set-hostname <siem-host>
sudo sed -i 's/<old-hostname>/<siem-host>/g' /etc/hosts
sudo systemctl stop wazuh-dashboard wazuh-manager wazuh-indexer
sudo rm -rf /var/run/wazuh-indexer.pid /var/ossec/var/run/*.pid
sudo systemctl start wazuh-indexer   # allowed ~45 seconds for database threads to settle
sudo systemctl start wazuh-manager wazuh-dashboard
```

### Result

Cluster communication was restored and the logging engine initialized cleanly. The dashboard came back online under the corrected hostname with log ingestion fully operational and **no data loss**.

---

## 5. Lessons and engineering takeaways

- **OS health is a prerequisite for security visibility.** A stalled package manager and a broken kernel driver can silently prevent an endpoint from enrolling in centralized monitoring.
- **Stateful services need ordered recovery.** Clearing stale locks and starting the indexer before the manager and dashboard prevents shard corruption and retry storms.
- **Risk acceptance must be explicit.** Some legacy CVEs in vendor or proprietary components are acceptable only when network isolation is strong and the decision is documented, not assumed.
- **Separate concurrent failures.** The dashboard's Axios errors and the endpoint's dpkg failure looked related but had different root causes; untangling them was the key diagnostic step.

---

## 6. Skills demonstrated

- Linux server administration: headless Ubuntu, `systemctl`, networking, package management
- SIEM operations: Wazuh manager, indexer, dashboard, agent enrollment, alert triage
- Vulnerability management: CVE analysis, Ubuntu Pro/ESM, forced package upgrades, documented risk acceptance
- Disaster recovery: root-cause analysis, lock-file cleanup, OpenSearch shard recovery, ordered service startup
- Diagnostic discipline: log correlation, hostname/DNS validation, separating concurrent failure modes
