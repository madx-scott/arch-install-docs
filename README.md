**Name:** Maddox Wise  
**Course:** CYB 3353  
**Date:** November 2025  
**VM Platform:** VMware Workstation Player (Windows)  
**Deliverable #1**

---

## 1. VM Setup
- Created a VM using **VMware Workstation Player** (20 GB disk, 2 GB RAM, 2 cores).  
- Attached the **archlinux-2025.11.01-x86_64.iso**.  
- Set the VM to use **UEFI firmware** by adding `firmware = "efi"` in the `.vmx` file.  
- **Issue:** Initially, Fusion couldn’t detect the bootloader.  
- **Fix:** Reattached the ISO and forced **EFI firmware** in the VM settings.

---

## 2. Boot & Network Setup
Once booting from the Arch Linux ISO:
bash
ping -c 1 archlinux.org
timedatectl set-ntp true
Verified network connectivity by pinging an external server.

Enabled NTP (Network Time Protocol) to sync system time.

3. Disk Partitioning (GPT)
Partitioned the disk using cfdisk:

bash
 
cfdisk /dev/sda
Created /dev/sda1 for EFI (500MB) and /dev/sda2 for root.

Formatted partitions:

bash
 
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
Mounted the partitions:

bash
 
mount /dev/sda2 /mnt
mkdir /mnt/boot && mount /dev/sda1 /mnt/boot
Note: Used cfdisk for simplicity in partitioning.

4. Base Installation
Installed the base system:

bash
 
pacstrap -K /mnt base linux linux-firmware vim networkmanager sudo git base-devel openssh
Generated the fstab file:

bash
 
genfstab -U /mnt >> /mnt/etc/fstab
Chrooted into the new system:

bash
 
arch-chroot /mnt
5. System Configuration
Set up the timezone, system clock, and locale:

bash
 
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
sed -i 's/^#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
Set the hostname:

bash
 
echo 'arch-vm' > /etc/hostname
6. Users & Sudo Configuration
Created user accounts and set up sudo permissions:

bash
 
passwd
useradd -m -G wheel maddox
useradd -m -G wheel codi
passwd maddox
passwd codi
Edited sudoers file:

bash
 
EDITOR=vim visudo  # uncomment %wheel ALL=(ALL:ALL) ALL
Enabled NetworkManager:

bash
 
systemctl enable NetworkManager
7. Bootloader Installation
Installed systemd-boot:

bash
 
bootctl install
Retrieved PARTUUID for the root partition:

bash
 
ROOTUUID=$(blkid -s PARTUUID -o value /dev/sda2)
Created a bootloader entry:

bash
 
cat >/boot/loader/entries/arch.conf <<EOF
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=$ROOTUUID rw
EOF
8. Desktop Environment Installation
Installed LXQt (lightweight desktop environment):

bash
 
pacman -S xorg-server sddm lxqt pipewire wireplumber
systemctl enable sddm
systemctl set-default graphical.target
9. Customizations
Installed zsh and set it as the default shell:

bash
 
pacman -S zsh
chsh -s /bin/zsh maddox
Added aliases to .zshrc:

bash
 
cat >> /home/maddox/.zshrc <<'EOF'
export CLICOLOR=1
alias ll='ls -alF'
alias gs='git status'
alias ..='cd ..'
EOF
Enabled SSH:

bash
 
systemctl enable sshd
10. AUR Helper Installation (yay)
Installed yay to handle AUR packages:

bash
 
sudo -u maddox bash -lc '  
  cd /tmp  
  git clone https://aur.archlinux.org/yay-bin.git  
  cd yay-bin  
  makepkg -si --noconfirm  
  yay -S --noconfirm google-chrome  
'
11. Final Verification
Verified network and user configuration:

bash
 
ip addr show | grep inet
getent group wheel
echo $SHELL
Confirmed that zsh is set as the default shell and both users are in the wheel group with sudo privileges.

12. Problems & Fixes
Problem	Fix
“No compatible boot loader found”	Forced EFI firmware in .vmx
Missing locale	Ran locale-gen
Black screen after install	Enabled sddm and graphical.target
No internet	Enabled NetworkManager service
