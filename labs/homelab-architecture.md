# Home Lab Architecture & Design Notes

A deliberately segmented, self-hosted lab built on repurposed hardware and virtual machines. This document describes the design *principles* and *trade-offs* behind the environment — not the operational layout. Hostnames, addresses, mount points, hardware identifiers, and physical locations are deliberately generalized or omitted.

The lab exists to practise the parts of infrastructure work that are hard to fake: segmentation, centralized observability, zero-trust access, virtualization, and recovery under failure.

---

## Design goals

1. **Segment by trust, not by convenience.** Untrusted clients, management traffic, and monitored hosts never share a flat network.
2. **No public-facing management ports.** Administrative access is overlay-only.
3. **Observability by default.** Every long-running host exposes metrics and logs before it is considered "done".
4. **Full lifecycle ownership.** From bare-metal install and partitioning to recovery from a corrupted service.
5. **Reproducibility.** Installs and rebuilds follow written, repeatable procedures rather than memory.

---

## Topology (generalized)

```text
                 [ Simulation node ]
                 Low-cost SBC · offensive tooling
                          │
                          │ synthetic recon / test traffic
                          ▼
        ┌───────────────────────────────────────┐
        │      Zero-trust overlay (mesh VPN)     │
        │  WireGuard · admin laptop · phone      │
        └───────────────────┬───────────────────┘
                            │ encrypted admin plane
              ┌─────────────┴─────────────┐
              │                           │
     [ Primary node ]              [ Secondary node ]
     Hypervisor + storage         Media / telemetry client
     SIEM · metrics · VMs         Metrics exporter
              │
     ┌────────┴─────────┐
     │                  │
[ Isolated client ]  [ Isolated guest / IoT ]
  zone · high metric   zone · no route to SIEM
```

---

## Network design

### Trust zones

| Zone | Purpose | Controls |
| :--- | :--- | :--- |
| **Management plane** | Administrative access to all hosts | Mesh VPN with node-level authorization; no WAN-exposed SSH |
| **Isolated client zone** | Household clients, media playback, legacy devices | Separate interface, high route metric, no internet path |
| **Monitored host zone** | SIEM, agents, observability stack | Reachable from the overlay only |
| **Guest / IoT zone** | Untrusted devices | Segmented; no route to the SIEM or admin plane |
| **Maintenance WAN** | Package updates, dependency pulls | Enabled on demand, then removed |

### Why a host firewall is not the primary control

The design relies on **physical and interface separation plus routing metrics**, not on a per-host software firewall. The reasoning:

- Segmentation is enforced at the boundary, so a single misconfigured host cannot expose the management plane.
- Fewer overlapping firewall rules means less configuration drift and fewer false assumptions about what is actually blocked.
- The trade-off is explicit: the boundary must be correct, because hosts do not carry a second line of defence. That is why the isolated zone has *no route* rather than a *blocked route*.

This is a deliberate risk decision, documented rather than accidental.

---

## Compute and storage tiering

The primary node separates storage by workload profile instead of pooling everything on one device:

| Tier | Holds | Why |
| :--- | :--- | :--- |
| **OS SSD** | Operating system, core packages, global configuration | Small, fast, and easy to rebuild |
| **State SSD** | Databases, service state, metadata caches | Isolates sustained random I/O from the OS |
| **Cache / VM SSD** | VM disk images, working caches, model weights | High-throughput, disposable, rebuildable |
| **Bulk HDD pool** | Media libraries, archives, static assets | Capacity over speed; not on the critical path |

The separation keeps a busy database from starving the hypervisor, and keeps large media reads off the disks that serve running services.

---

## Virtualization

Two frameworks are used on purpose, because they solve different problems:

### KVM / QEMU with libvirt — performance tier

- Hardware-accelerated guests for workloads that need near-native throughput.
- Storage on the fast SSD tier.
- Used for isolated desktop workloads where GPU or I/O performance matters.

### VirtualBox — compatibility and sandbox tier

- Exact legacy environment emulation for operating-system testing.
- Sandboxes for rolling-release and gaming-oriented distributions.
- Legacy Windows guests for protocol and compatibility testing on the isolated network.
- Reproducible snapshots that make destructive testing safe.

The split is intentional: performance workloads get KVM, compatibility workloads get VirtualBox, and neither contaminates the other.

---

## Observability stack

Monitoring runs as native `systemd` services rather than container wrappers, keeping the path from host to dashboard short and debuggable.

```text
[ Primary node ]        [ Secondary node ]
 node_exporter           node_exporter
      │                       │
      └───────────┬───────────┘
                  ▼   scrape over LAN
            Prometheus
            (time-series store)
                  │
                  ▼
             Grafana
             (dashboards)
                  │
                  ▼
        Remote admin via overlay
```

| Component | Role |
| :--- | :--- |
| `node_exporter` | Exposes CPU, memory, disk I/O, and network metrics per host |
| `prometheus` | Scrapes and stores time-series metrics on a fixed interval |
| `grafana` | Renders dashboards; bound to the host and reached over the overlay |

The same overlay that carries administration also carries dashboards, so there is no separate public monitoring endpoint to secure.

---

## Supporting services

- **Local LLM inference** — a small model served locally with a browser front-end, used for offline experimentation and to keep prompts on-premises. Model weights live on the fast SSD tier.
- **Isolated chat service** — a lightweight text service bound to the isolated zone for host-to-host coordination without external dependencies.
- **Zero-authentication media server** — local streaming for trusted clients on the isolated switch, with metadata cached for offline browsing. External access is deliberately absent.
- **Paused file share** — a legacy ROM/file share is intentionally offline while its storage layout is re-evaluated; the rest of the stack is unaffected by design.

---

## Security decisions and trade-offs

| Decision | Rationale | Trade-off accepted |
| :--- | :--- | :--- |
| Overlay-only admin access | Removes WAN-exposed management ports and works behind CGNAT | Admin depends on the overlay being healthy |
| No host firewall as primary control | Segmentation at the boundary, less rule drift | Boundary correctness is critical |
| Isolated zone with no route | Removes lateral movement by construction | Some conveniences require physical moves |
| Native services over containers | Shorter debug path, fewer abstractions | Less portable than containerized stacks |
| Local LLM instead of cloud | Prompts and data stay on-premises | Smaller models, lower capability |
| Two virtualization frameworks | Right tool per workload | More to maintain and document |

---

## Lessons learned

- **Segment before you secure.** A flat network makes every later control harder to reason about. Designing trust zones first made the remaining decisions obvious.
- **Write the recovery path before you need it.** Every service that mattered eventually failed once; the ones with a written procedure recovered fastest.
- **Prefer "no route" to "blocked route".** A path that does not exist cannot be misconfigured open.
- **Document risk acceptance.** Isolating a host is a valid mitigation, but only if the reasoning is written down where the next person can find it.
- **Rebuildability is a security property.** If an install cannot be reproduced from notes, it cannot be trusted to recover.

---

## Redaction note

This document intentionally omits hostnames, IP addresses, subnets, mount points, hardware model numbers, physical locations, account names, and tokens. The full operational blueprint is kept private; this page is the shareable design summary.
