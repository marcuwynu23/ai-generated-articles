
# 🐧 How to Create Your Own Custom Linux Distribution (Arch & Debian Based)

Have you ever wanted to build your own Linux distribution — one that boots into your favorite tools, is free from bloatware, and carries your branding? Whether for developers, schools, or personal experiments, creating a custom Linux distro is not only possible but also straightforward with the right tools.  

In this article, we’ll explore how to build a custom distro using **Arch Linux** and **Debian**, two of the most popular Linux bases.  

---

## 🔹 Why Create a Custom Linux Distro?
- **Developer-focused OS**: Ship preinstalled dev tools (Docker, Git, Nmap, Node.js, Python, etc.)  
- **Lightweight system**: Remove unnecessary packages and bloat  
- **Branding**: Add custom wallpapers, logos, welcome banners (`motd`, `issue`)  
- **Live USB boot**: Distribute as an ISO for testing or real-world use  
- **Learning**: Understand the Linux boot process and system internals  

---

# 1. Building on Arch Linux (Using `archiso`)

Arch Linux is known for being minimal and rolling-release. It’s perfect for building a lean, developer-focused distro.

### 🛠 Requirements
- An Arch Linux environment (VM, container, or physical machine)  
- `archiso` package  

Install the tool:
```bash
sudo pacman -Sy archiso
```

### 📂 Setup Project
```bash
cp -r /usr/share/archiso/configs/releng peculiarlinux
cd peculiarlinux
```

This copies the official ArchISO template into your workspace.

### 📦 Add Packages
Edit `packages.x86_64` and add your desired software:
```txt
base
linux
linux-firmware
sudo
openssh
git
nmap
htop
docker
containerd
docker-compose
vim
neovim
python
nodejs
npm
rust
go
xorg
xfce4
lightdm
lightdm-gtk-greeter
```

### 👤 Add Default User
Create `airootfs/usr/local/bin/create-default-user.sh`:
```bash
#!/bin/bash
USERNAME=developer
if ! id "$USERNAME" &>/dev/null; then
  useradd -m -G wheel,docker -s /bin/bash "$USERNAME"
  echo "$USERNAME:peculiar" | chpasswd
  echo "%wheel ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/90-wheel-nopasswd
fi
```

Make it executable:
```bash
chmod +x airootfs/usr/local/bin/create-default-user.sh
```

Add a systemd service (`airootfs/etc/systemd/system/create-default-user.service`) to run it on boot.

### 🎨 Branding
- MOTD: `airootfs/etc/motd`  
- Issue banner: `airootfs/etc/issue`  
- Wallpapers: `airootfs/usr/share/backgrounds/peculiar/`  

### 💿 Build ISO
```bash
mkarchiso -v -o ../out .
```

Result → `peculiarlinux-YYYY.MM.DD-x86_64.iso`  
This ISO is hybrid, so you can boot it in VMs or write it directly to USB.

---

# 2. Building on Debian (Using `debootstrap` + `refractasnapshot`)

Debian is a stable and clean base with fewer extras than Ubuntu, making it a great choice for a minimal but reliable distro.

### 🛠 Requirements
```bash
sudo apt update
sudo apt install debootstrap squashfs-tools grub-pc-bin grub-efi-amd64-bin xorriso live-boot refractasnapshot -y
```

### 📂 Bootstrap Minimal Debian
```bash
sudo debootstrap --arch=amd64 bookworm ./peculiar-chroot http://deb.debian.org/debian/
```

This creates a base Debian system inside `./peculiar-chroot`.

### 🔧 Enter Chroot
```bash
sudo mount --bind /dev ./peculiar-chroot/dev
sudo mount --bind /proc ./peculiar-chroot/proc
sudo mount --bind /sys ./peculiar-chroot/sys
sudo chroot ./peculiar-chroot /bin/bash
```

### 📦 Install Packages
Inside chroot:
```bash
apt update
apt install -y sudo git nmap openssh-server docker.io docker-compose   python3 python3-pip nodejs npm golang htop tmux curl wget   xorg xfce4 lightdm lightdm-gtk-greeter
```

### 👤 Add Default User
```bash
useradd -m -G sudo,docker -s /bin/bash developer
echo "developer:peculiar" | chpasswd
echo "%sudo ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/90-sudo-nopasswd
```

### 🎨 Branding
- `/etc/motd` for login message  
- `/etc/issue` for pre-login banner  
- `/usr/share/backgrounds/peculiar/` for wallpapers  

### ❌ Exit Chroot & Cleanup
```bash
exit
sudo umount ./peculiar-chroot/{dev,proc,sys}
```

### 💿 Build ISO
Run:
```bash
sudo refractasnapshot
```

This generates a bootable ISO, typically named `live-image-amd64.hybrid.iso`.

---

# 3. Which Base Should You Choose?

| Feature | Arch (archiso) | Debian (debootstrap/live-build) |
|---------|----------------|--------------------------------|
| Release model | Rolling release | Stable release |
| ISO size | Very small, minimal base | Slightly bigger base |
| Ease of updates | Always latest | Needs backports for latest packages |
| Learning curve | More DIY, teaches Linux internals | Easier to automate |
| Best for | Bleeding-edge dev distro | Stable, long-term live distro |

---

# 🎯 Conclusion

Creating your own Linux distribution is not as complex as it sounds.  
- With **ArchISO**, you get bleeding-edge minimalism and maximum control.  
- With **Debian + debootstrap**, you get rock-solid stability and a clean base without Ubuntu’s extras.  

Both paths let you build a distro like **PeculiarLinux** — lightweight, branded, developer-ready, and bootable on USB or VMs.  
