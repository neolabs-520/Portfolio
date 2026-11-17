# Systems Administration Lab & Architecture Report
## Subject: Resilient Arch Linux Deployment & Virtualization Stabilization
**Author:** DevOps & Systems Administration Portfolio
**Date:** October 2026
**Document Class:** Technical Lab Manual & Infrastructure Blueprint

---

## Executive Summary
This document serves as an exhaustive deployment blueprint and academic lab manual for provisioning a stable, resilient **Arch Linux** environment within a virtualized hypervisor infrastructure. 

Born out of empirical failure analysis—specifically diagnosing and mitigating disk saturation bugs ($100\%$ root filesystem utilization) and graphical rendering race conditions under default display pipelines—this guide outlines the exact tactical methodology required to achieve a clean, functional **GNOME Desktop environment** utilizing a forced **X11 display infrastructure**.

---

## Section 1: Host Hypervisor Provisioning & Virtual Hardware Target

Improper hardware allocation at the hypervisor level is the primary catalyst for down-stream operating system failures during bootstrapping and major package ingestion phases. The target specifications below are hard requirements engineered to completely isolate the installation from resource starvation.

### 1.1 Compute & Memory Allocation
* **System Memory (RAM):** **4096 MB (4 GB)** minimum. Allocating less than 4 GB triggers high swap dependency, severe performance degradation, and potential Out-Of-Memory (OOM) kernel panics during the graphical desktop compilation phases.
* **Processor Topology:** Minimum **2 vCPUs**. Single-core virtual machines introduce CPU scheduling bottlenecks when handling simultaneous display manager initializations and background network configurations.

### 1.2 Storage Architecture
* **Virtual Disk Format:** VirtualBox Disk Image (`.vdi`), configured with **Dynamically Allocated Storage**.
* **Minimum Capacity Provision:** **40 GB**. 
  * *Failure Analysis Insight:* Initial deployments bounded by standard 15 GB virtual disks experience total root file system saturation ($100\%$ capacity blocks) during bulk package downloads via `pacman`. This completely breaks administrative write privileges, corrupts the `sudoers` structural identity, and causes essential system daemons like `gnome-settings-daemon` to terminate unexpectedly. A 40 GB baseline guarantees an operational safety margin.
* **Controller Type:** SATA Interface with **Host I/O Cache Enabled**. This optimizes multi-threaded disk IO operations during initial package unpacking.

### 1.3 Display Layer Configuration
* **Graphics Controller Type:** **VMSVGA** (The standard optimized driver framework for Linux guests inside VirtualBox).
* **Video Memory (VRAM):** **128 MB** (Fully maxed out).
* **Acceleration Pipeline:** **3D Acceleration Checked**. Failure to explicitly activate 3D hardware acceleration forces software rasterization via LLVMpipe, severely choking GNOME's compositor interface.

---

## Section 2: Storage Layer Partitioning & Filesystem Initialization

A clean, modern system layout demands standard GUID Partition Table (GPT) alignment. This eliminates the logical limitations of legacy MBR sector flags and positions the environment cleanly for Unified Extensible Firmware Interface (UEFI) handshake paths.

### 2.1 Disk Topology Engineering
Using an interactive partition engine (`cfdisk /dev/sda`), wipe the drive structures and implement a dedicated two-partition schema:

| Partition Target | Sector Start/End | Capacity Allocation | Filesystem Mapping | Mount Point |
| :--- | :--- | :--- | :--- | :--- |
| `/dev/sda1` | Standard Alignment | **1.0 GB** | FAT32 (`vfat`) | `/boot/efi` |
| `/dev/sda2` | Remainder of Disk | **39.0+ GB** | Ext4 Journaled | `/` (Root) |

### 2.2 Filesystem Initialization Math & Mechanics
Initialize the storage layers cleanly from the live environment prompt.

```bash
# Initialize the EFI System Partition (ESP) using standard FAT32 allocation
mkfs.fat -F 32 /dev/sda1

# Initialize the Linux Main Root Layer with standard ext4 block journaling
mkfs.ext4 /dev/sda2
```

### 2.3 Structural Mounting Sequence
Mount the primary root container prior to mapping the underlying firmware boot interface:

```bash
# Establish the root mount mountpoint
mount /dev/sda2 /mnt

# Generate the explicit subdirectory and bind the EFI partition
mount --mkdir /dev/sda1 /mnt/boot/efi
```

---

## Section 3: Package Bootstrapping & Core System Commodity Ingestion

To insulate the system from post-reboot isolation—a state where a kernel boots but cannot form network hooks, render input interfaces, or access basic admin tools—the initial `pacstrap` deployment must include foundational software commodities.

```bash
pacstrap -K /mnt base linux linux-firmware nano sudo networkmanager virtualbox-guest-utils xorg-server gnome gnome-tweaks
```

### 3.1 Architectural Package Analysis
* `base`, `linux`, `linux-firmware`: Form the minimal structural kernel and device support framework.
* `nano`, `sudo`: Establish instant baseline text editing capability and cryptographic administrative access filters.
* `networkmanager`: Eradicates manual link assignment; automates link state machines upon first system bootstrap.
* `virtualbox-guest-utils`: Ingests the required kernel-level display and sync hooks to interface smoothly with the VMSVGA hypervisor architecture.
* `xorg-server`, `gnome`, `gnome-tweaks`: Aggregates the fundamental graphical server, desktop ecosystem, and UI configuration dials into a single atomic installation block.

---

## Section 4: Post-Chroot Micro-Architectural Configurations

With the base binaries mapped inside the host directory structure, translate the runtime environment inward to configure structural system values.

```bash
# Generate a clean system storage architecture map tracking explicit UUID flags
genfstab -U /mnt >> /mnt/etc/fstab

# Cross the directory threshold into the internal native system layer
arch-chroot /mnt
```

### 4.1 Chronological Synchronization & Localization
```bash
# Bind the internal timezone structure explicitly to Universal Coordinated Time (UTC)
ln -sf /usr/share/zoneinfo/UTC /etc/localtime
hwclock --systohc

# Generate system-wide linguistic structures
# Note: Ensure "en_US.UTF-8 UTF-8" is completely uncommented inside /etc/locale.gen prior to execution
nano /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Establish unique hostname node identifier inside the network
echo "<hostname>" > /etc/hostname
```

### 4.2 Core Identity Lockout Strategy
Prevent unauthorized root administrative exploits by establishing a strong, non-blank root security token:
```bash
passwd
```

---

## Section 5: Secure Permission Layering & User Creation

To prevent localized identity degradation—where a standard user account loses administrative rights due to an incomplete or unwritten `sudoers` database—the primary identity must be built sequentially.

```bash
# Provision the primary user, establish default home profiles, and append to group 'wheel'
useradd -m -G wheel -s /bin/bash <username>

# Set the secure password token for '<username>' immediately
passwd <username>
```

### 5.1 Hardcoding Sudoers Access Permissions
Do not rely on unstable or unwritten shell environmental variables. Safely unlock system-wide root privilege elevation access routines:
```bash
EDITOR=nano visudo
```
Locate the exact block tracking the `wheel` infrastructure group. Completely strip away the comment symbol (`#`) to hardwire structural administration access:
```text
%wheel ALL=(ALL:ALL) ALL
```

---

## Section 6: Display Pipeline Stabilization (The X11 Fail-Safe)

By default, modern GNOME deployments leverage the Wayland display server protocol. However, inside VirtualBox environments running hardware-abstracted VMSVGA graphics drivers, Wayland routinely enters an unstable execution cycle. This results in frozen desktop layouts, random mouse drops, and unexpected `gnome-settings-daemon` failures. 

To achieve industrial stability, the display layer must be forced backwards into a legacy **X11 pipeline execution matrix**.

```bash
# Modify the Display Manager runtime behavior matrix
nano /etc/gdm/custom.conf
```
Locate the following configuration declaration inside the `[daemon]` block. Remove the comment character (`#`) to forcefully deactivate Wayland system-wide:
```ini
WaylandEnable=false
```

### 6.1 Automating Essential System Daemons
Set the display manager, network controller, and hypervisor guest integration hooks to spin up natively on system initialization:
```bash
systemctl enable NetworkManager.service
systemctl enable gdm.service
systemctl enable vboxservice.service
```

---

## Section 7: Orderly Environmental De-provisioning & First Boot Validation

Improper termination of the installer wrapper can cause partial partition map corruption. Execute an orderly shutdown of the environment:

```bash
# Escape the chroot environment layer back into the live media wrapper
exit

# Recursively unmount all mapped directories and partition boundaries safely
umount -R /mnt

# Cold-reboot the system hardware container
reboot
```

### 7.1 First-Run Post-Mortem Diagnostics
Upon navigating past the graphical GDM login portal and logging into the desktop workspace, immediately run diagnostics within a local terminal to confirm operational integrity:

1. **Storage Cushion Review:** `df -h /` must reflect ample unallocated space out of the new 40 GB storage array.
2. **Pacman Maintenance Cleanup:** Reclaim gigabytes of safety space by clearing obsolete installer cache objects:
   ```bash
   sudo pacman -Sc --noconfirm
   ```
3. **Environment Audit:** `echo $XDG_SESSION_TYPE` should return `x11`, confirming the successful Wayland bypass strategy.

---
### Academic Disclaimer
*This lab guide is published for systems engineering portfolio review. The exercises contained within demonstrate automated provisioning logic, failure vector analysis, and hypervisor graphics stabilization workflows.*
