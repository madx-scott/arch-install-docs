Overview

This document details the process of installing and configuring Arch Linux ARM inside a UTM virtual machine on macOS. The installation was done manually, without the archinstall command, and includes all required modifications such as a GUI desktop environment, user creation, alternate shell, SSH setup, terminal color, and custom aliases.

Step 1 – Setup in macOS and UTM
1. Download Required Tools

UTM: https://mac.getutm.app

Archboot AArch64 ISO: https://release.archboot.com/aarch64/latest/iso/

2. Create a New Virtual Machine

Open UTM → Create New Virtual Machine.

Select Virtualize → Linux.

System settings:

Architecture: ARM64 (aarch64)

Memory: 4096 MB

Storage: 20 GB (QCOW2 format)

Boot ISO: select the downloaded Archboot ISO

Network: NAT

Start the VM and boot into the Arch ISO.

Step 2 – Base Installation (Manual Method)
Partition the Disk
fdisk /dev/vda


Create a GPT table.

Partition 1: 512M EFI (type EF00)

Partition 2: remaining space as root.

Format and Mount
mkfs.fat -F32 /dev/vda1
mkfs.ext4 /dev/vda2
mount /dev/vda2 /mnt
mkdir /mnt/boot
mount /dev/vda1 /mnt/boot

Install the Base System
pacstrap /mnt base linux linux-firmware vim sudo networkmanager
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt

Step 3 – System Configuration
Set Time and Hostname
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
echo "archarm" > /etc/hostname

Enable Networking
systemctl enable NetworkManager

Set Root Password
passwd

Install and Configure Bootloader
pacman -S grub efibootmgr
grub-install --target=arm64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

Step 4 – User Creation and Sudo Access

Create users:

useradd -m -G wheel maddox
passwd maddox
useradd -m -G wheel codi
passwd codi


Grant sudo permissions:

visudo


Uncomment:

%wheel ALL=(ALL) ALL

Step 5 – Install Desktop Environment

LXQt is lightweight and works well for ARM.

pacman -S xorg lxqt sddm
systemctl enable sddm


Reboot to test the graphical login.

Step 6 – Alternate Shell and Aliases

Install and configure zsh:

pacman -S zsh
chsh -s /bin/zsh maddox


Edit ~/.zshrc:

alias ll='ls -lah --color=auto'
alias update='sudo pacman -Syu'
alias cls='clear'


Enable color in package manager:

sed -i 's/#Color/Color/' /etc/pacman.conf

Step 7 – SSH Setup

Install and enable SSH for remote access:

pacman -S openssh
systemctl enable sshd
systemctl start sshd


Verify IP:

ip a

Step 8 – Boot and Verification

Exit the chroot environment:

exit
umount -R /mnt
reboot


After reboot, verify the following:

whoami
sudo whoami
lsb_release -a
ip a


Check that the system boots to the LXQt desktop and the network connects automatically.

Step 9 – Browser and AUR Setup

Install Firefox for the GUI:

sudo pacman -S firefox


Install git and base-devel for AUR access:

sudo pacman -S git base-devel


Install yay (AUR helper):

git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si


Example AUR package installation:

yay -S neofetch
neofetch

Step 10 – Problems Encountered and Solutions
Problem	Description	Solution
EFI bootloader failed to install	GRUB did not detect EFI correctly	Reinstalled using --target=arm64-efi and recreated config
No network connection after reboot	NetworkManager was not enabled	Enabled and started NetworkManager service
Sudo permission denied	Wheel group not active	Uncommented %wheel ALL=(ALL) ALL in /etc/sudoers
No terminal color output	Defaults were off	Enabled color in /etc/pacman.conf and shell aliases
