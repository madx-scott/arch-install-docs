# arch-install-docs


Arch Linux Installation Documentation
Name: Maddox Wise
Course: CYB 3353
Date: November 2025
VM Platform: VMware Fusion (Mac, UEFI)
Deliverable #1

1. VM Setup

Created VM in VMware Fusion (20 GB disk, 2 GB RAM, 2 cores).

Attached archlinux-2025.11.01-x86_64.iso.

Fixed “No compatible boot loader found” by setting firmware = "efi" in the .vmx file.

Issue: Fusion couldn’t find the OS at first. Fix: Reattached ISO and forced EFI firmware.

2. Boot & Network
ping -c1 archlinux.org  
timedatectl set-ntp true  


Verified internet and synchronized time.

3. Disk Partitioning (GPT)
cfdisk /dev/sda  
mkfs.fat -F32 /dev/sda1  
mkfs.ext4 /dev/sda2  
mount /dev/sda2 /mnt  
mkdir /mnt/boot && mount /dev/sda1 /mnt/boot  


Used cfdisk for partitioning (easier to visualize).

4. Base Install
pacstrap -K /mnt base linux linux-firmware vim networkmanager sudo git base-devel openssh  
genfstab -U /mnt >> /mnt/etc/fstab  
arch-chroot /mnt  


Installed the base system and entered the chroot environment.

5. System Configuration
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime  
hwclock --systohc  
sed -i 's/^#en_US.UTF-8/en_US.UTF-8/' /etc/locale.gen  
locale-gen  
echo 'LANG=en_US.UTF-8' > /etc/locale.conf  
echo 'arch-vm' > /etc/hostname  


Set time zone, locale, and hostname.

6. Users & Sudo
passwd  
useradd -m -G wheel maddox  
useradd -m -G wheel codi  
passwd maddox  
passwd codi  
EDITOR=vim visudo  # uncomment %wheel ALL=(ALL:ALL) ALL  
systemctl enable NetworkManager  


Created users and enabled sudo access.

7. Bootloader
bootctl install  
ROOTUUID=$(blkid -s PARTUUID -o value /dev/sda2)  
cat >/boot/loader/entries/arch.conf <<EOF  
title   Arch Linux  
linux   /vmlinuz-linux  
initrd  /initramfs-linux.img  
options root=PARTUUID=$ROOTUUID rw  
EOF  


Installed bootloader, fixed PARTUUID issue.

8. Desktop Environment
pacman -S xorg-server sddm lxqt pipewire wireplumber  
systemctl enable sddm  
systemctl set-default graphical.target  


Installed LXQt for a lightweight desktop environment.

9. Customizations
pacman -S zsh  
chsh -s /bin/zsh maddox  
cat >> /home/maddox/.zshrc <<'EOF'  
export CLICOLOR=1  
alias ll='ls -alF'  
alias gs='git status'  
alias ..='cd ..'  
EOF  


Set up zsh, colorized prompt, and SSH access.

10. AUR Setup
sudo -u maddox bash -lc '  
  cd /tmp  
  git clone https://aur.archlinux.org/yay-bin.git  
  cd yay-bin  
  makepkg -si --noconfirm  
  yay -S --noconfirm google-chrome  
'  


Installed AUR helper (yay) and Google Chrome.

11. Verification
ip addr show | grep inet  
getent group wheel  
echo $SHELL  


Verified networking, sudo users, and default shell.

12. Problems & Fixes
Problem	Fix
“No compatible boot loader found”	Forced EFI firmware in .vmx
Missing locale	Ran locale-gen
Black screen after install	Enabled sddm and graphical.target
No internet	Enabled NetworkManager service
