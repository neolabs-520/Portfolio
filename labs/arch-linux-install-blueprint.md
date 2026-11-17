# ARCH LINUX VM INSTALLATION BLUEPRINT: CLEAN SLATE
This blueprint was forged from trial, error, and hard-earned experience. It ensures a bulletproof, properly allocated, and perfectly stable Arch Linux system running GNOME over X11 inside VirtualBox.

## SECTION 1: HOST VM SPECIFICATIONS (PRE-FLIGHT)
Before mounting the ISO, configure your VirtualBox Manager exactly as follows to prevent resource bottlenecks or disk saturation:
* **System Memory:** 4096 MB (4 GB) minimum.
* **Processors:** 2 vCPUs minimum.
* **Graphics Controller:** `VMSVGA` with **3D Acceleration enabled** and 128 MB VRAM.
* **Storage Interface:** SATA Controller with "Host I/O Cache" enabled.
* **Virtual Disk (`.vdi`):** **40 GB dynamically allocated** minimum (prevents the 100% disk exhaustion trap encountered previously).

---

## SECTION 2: DISK MANAGEMENT & PARTITIONING (`sda`)
Ensure the partition table is clean and the layout is fully aligned before bootstrapping packages.

### 1. Partition Creation (`cfdisk /dev/sda`)
Select a **GPT** partition table format and allocate as follows:
* **`/dev/sda1`**: 1 GB EFI System Partition.
* **`/dev/sda2`**: Remaining space (39+ GB) for the Linux Root File System.

### 2. Filesystem Initialization
```bash
# Format the EFI System Partition
mkfs.fat -F 32 /dev/sda1

# Format the Main Root Partition
mkfs.ext4 /dev/sda2
```

### 3. Mounting the Layout
```bash
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot/efi
```

---

## SECTION 3: PACSTRAP & SYSTEM COMMODITIES
To prevent post-reboot isolation or missing system configurations, run the `pacstrap` command with all essential utilities pre-cached:

```bash
pacstrap -K /mnt base linux linux-firmware nano sudo networkmanager virtualbox-guest-utils xorg-server gnome gnome-tweaks
```

---

## SECTION 4: POST-CHROOT INITIALIZATION
Generate the filesystem architecture map and enter the environment to finish setup.

```bash
# Generate fstab explicitly tracking UUIDs
genfstab -U /mnt >> /mnt/etc/fstab

# Chroot into your new system
arch-chroot /mnt
```

### 1. Localization & Network Handshake
```bash
# Set timezone to UTC
ln -sf /usr/share/zoneinfo/UTC /etc/localtime
hwclock --systohc

# Generate locales (Uncomment en_US.UTF-8 UTF-8 in /etc/locale.gen first)
nano /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Set system identity
echo "<hostname>" > /etc/hostname
```

### 2. Root Security Checkpoint
```bash
# Define the root user's password
passwd
```

---

## SECTION 5: FILESYSTEM PERMISSIONS & USER CONFIGURATION
To ensure your user identity has undisputed administrative access and won't get locked out by permission degradation:

```bash
# Create user '<username>' with proper supplementary groups and bash shell
useradd -m -G wheel -s /bin/bash <username>

# Set user '<username>' password immediately
passwd <username>
```

### Sudoers Activation
Open the configuration safely with nano:
```bash
EDITOR=nano visudo
```
Locate and **completely uncomment** the exact line below (remove the `#`):
```text
%wheel ALL=(ALL:ALL) ALL
```

---

## SECTION 6: DISPLAY STABILIZATION & SERVICES (ANTI-CRASH)
Wayland sessions can render blank layouts or cause components to fail inside VirtualBox graphics pipelines. We force an X11 environment as a fail-safe.

### 1. Disable Wayland in GDM
```bash
# Open GDM layout configuration file
nano /etc/gdm/custom.conf
```
Locate and **completely uncomment** the following configuration declaration:
```ini
WaylandEnable=false
```

### 2. Initialize Core Services
Force your display infrastructure, network handlers, and hypervisor drivers to trigger automatically on boot:
```bash
systemctl enable NetworkManager.service
systemctl enable gdm.service
systemctl enable vboxservice.service
```

---

## SECTION 7: SAFE REBOOT PROCEDURES
```bash
# Exit the chroot bubble
exit

# Unmount all system hooks cleanly
umount -R /mnt

# Reboot the virtual hardware
reboot
```