# Complete Guide to Installing Proxmox VE with ZFS on Your Server

Proxmox VE (Virtual Environment) is a powerful open-source platform for virtualization, combining KVM (virtual machines) and LXC (containers) into a single management interface. With support for ZFS, clustering, backups, and high availability, it’s an excellent choice for homelabs and enterprise environments alike.

This article walks you through the entire installation process — from downloading the ISO to configuring ZFS storage and network settings.

## 1. Download the Proxmox VE ISO

1. Visit the official download page:
   👉 Proxmox VE Downloads

2. Select the correct ISO for your server’s architecture:

- amd64 (x86_64) → Most Intel/AMD servers (default).

- arm64 → For ARM-based systems only.

3. Download the latest release, e.g.:

```sh
proxmox-ve_8.2-iso.iso
```

## 2. Create a Bootable USB

On a Windows machine:

1. Insert a USB stick (≥4 GB, all data will be erased).

2. Download Rufus
3. Configure Rufus:

- Device → Your USB stick

- Boot selection → Proxmox ISO

- Partition scheme →

  - GPT + UEFI (recommended)

  - MBR + BIOS (for legacy servers)

4. Click Start and wait for the process to complete.

## 3. Prepare Server BIOS/UEFI

1. Plug in your Proxmox USB.

2. Boot into BIOS/UEFI and check:

- Enable UEFI boot (recommended).

- Enable Intel VT-x / AMD-V (hardware virtualization).

- Enable VT-d / AMD-Vi (IOMMU) for PCI passthrough.

3. Choose the USB device in the boot menu (keys vary: F9, F11, F12, ESC, DEL).

## 4. Start the Proxmox Installer

When the Proxmox boot menu appears:

- Select Install Proxmox VE.

- Accept the EULA.

## 5. Configure Storage (ZFS Setup)

Choose where to install Proxmox VE.

## Recommended ZFS Configurations:

- ZFS RAID1 (mirror) → 2 disks for redundancy.

- ZFS RAID10/RAIDZ1/RAIDZ2 → 4+ disks.

- ext4 → Simple but lacks ZFS features.

⚠️ Proxmox will create an rpool ZFS pool for the system. Extra disks can be added later for VM storage.

## 6. Set Region and Time

- Country → Select your region.

- Timezone → Local timezone.

- Keyboard → Layout preference.

## 7. Create Root Credentials

- Set a root password (used for web login).

- Provide an email (for system notifications).

## 8. Configure Network

Select your main network interface and configure:

- Hostname (FQDN) → e.g. pve1.lab.local

- IP Address → Static (e.g. 192.168.1.90)

- Netmask → /24 or 255.255.255.0

- Gateway → Your router/firewall IP (e.g. 192.168.1.1)

- DNS Server → 1.1.1.1 (Cloudflare) or 8.8.8.8 (Google)

⚠️ The hostname must be a valid FQDN (hostname.domain).

## 9. Complete Installation

- Review your settings.

- Proxmox will partition disks, install the OS, and set up ZFS.

- Remove the USB when prompted and reboot.

## 10. Access the Proxmox Web Interface

1. After reboot, log into the server console to confirm network connectivity.

2. From your browser, access:

```sh
https://<your-server-ip>:8006
```

3. Login credentials:

- User → root

- Password → (the one you set during install)

## 11. Post-Installation Setup

Update the system:

```sh
apt update && apt full-upgrade -y
```

Switch to no-subscription repository:

```sh
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
apt update

```

Add extra disks (example: /dev/sdc + /dev/sdd)

```sh
wipefs -a /dev/sdc
wipefs -a /dev/sdd
zpool create vmdata mirror /dev/sdc /dev/sdd
pvesm add zfspool vmdata -pool vmdata -content images,rootdir

```

Now vmdata will appear as a storage pool in Proxmox for your VMs and containers.

- Proxmox VE installed on ZFS RAID1 (rpool).

- Web UI accessible at https://<IP>:8006.

- Extra disks available as vmdata storage pool.

- System is ready for VM and container deployment.
