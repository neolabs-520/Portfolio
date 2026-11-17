# Arch Linux Installation Journey: Manual Install and Desktop Recovery

## Overview

A manual installation of **Arch Linux** inside **VirtualBox**, built from the command line rather than an installer script. The first attempt surfaced three failures — disk exhaustion, a Wayland/graphics conflict, and a sudo permissions lockout — each of which was diagnosed and fixed. The lessons were then codified into a repeatable installation blueprint.

---

## Mission profile

- **Host hypervisor:** VirtualBox
- **Target OS:** Arch Linux (x86_64)
- **Goal:** A manually provisioned base system running a stable GNOME desktop

---

## Phase 1 — First install and failures

### 1. Disk exhaustion (100% root utilization)

**Cause:** The initial 15 GB virtual disk was too small for a full graphical desktop and its package cache.

**Symptoms:** The root partition (`/dev/sda2`) filled to 100%. `gnome-settings-daemon` and X11 display components crashed on login, dropping the system to a blank layout. `sudo` began failing because there was no space left to write.

**Fix:** Expanded the virtual disk (`.vdi`) in VirtualBox, then grew the partition from inside the guest:

```bash
cfdisk /dev/sda      # extend the sda2 partition boundary
partprobe            # re-read the partition table
resize2fs /dev/sda2  # grow the ext4 filesystem to fill the partition
```

### 2. Display server instability (Wayland vs. VirtualBox graphics)

**Symptom:** Intermittent display lockups and unrendered window components after login.

**Cause:** Wayland compositors conflicted with the virtualized `VMSVGA` graphics driver under this hypervisor configuration.

**Fix:** Forced GDM to use X11 instead of Wayland by setting `WaylandEnable=false` in `/etc/gdm/custom.conf`.

### 3. Sudo privileges and permissions lockout

**Cause:** Interrupted disk states during install left `/etc/sudoers` validation incomplete.

**Lesson:** Verify the `wheel` group configuration and run `visudo` before the final chroot unmount — not after the first reboot.

---

## Phase 2 — Lessons converted into a repeatable blueprint

Every failure on the first machine became a hard rule in the blueprint:

| Failure encountered | Correction built into the blueprint |
| :--- | :--- |
| 15 GB disk starvation | Standardize on **40 GB dynamically allocated** storage |
| Wayland graphical crashes | Enforce an X11 fallback in `/etc/gdm/custom.conf` immediately after install |
| Broken user permissions | Strict ordering of `useradd`, password setup, and `visudo` confirmation before leaving chroot |
| Pacman cache saturation | Add `pacman -Sc` cache cleanup as standard post-setup hygiene |

---

## Outcome and takeaways

The installation was recovered to a stable GNOME desktop over X11, and the failure analysis was captured in a reusable procedure.

This project demonstrates:

- **Storage and volume management** — resizing a live partition and filesystem, and understanding the boundary between the virtual disk, partition table, and filesystem.
- **Linux graphics pipelines** — debugging the display server stack (Wayland/Xorg) and the display manager (GDM).
- **Degraded-mode diagnostics** — reading logs and isolating faults while the system is partially broken and administrative tools are failing.

The full repeatable procedure is documented in [the installation blueprint](arch-linux-install-blueprint.md) and [the systems administration lab guide](arch-linux-sysadmin-lab.md).
