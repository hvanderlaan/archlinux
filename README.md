Installing ArchLinux 2014.10.01
===============================
This howto will let you install Arch Linux 2014.10.01 on vmware workstation 10

Downloading image
=================
```
wget 'http://arch.apt-get.eu/iso/2014.10.01/archlinux-2014.10.01-dual.iso'
```
Installing Arch Linux: Prepare the system
=========================================
First create a partition table on /dev/sda
```
fdisk /dev/sda
	- /dev/sda1 (512MB Bootable linux)
	- /dev/sda2 (25.5G linux)
	- /dev/sda3 (4G linux swap)
```
Create a swap partition
```
mkswap /dev/sda3
swapon /dev/sda3
```
Format /dev/sda[12] to ext4
```
mkfs.ext4 /dev/sda1
mkfs.ext4 /dev/sda2
```
Mount /dev/sda[12] to /mnt and /mnt/boot
```
mount /dev/sda2 /mnt
mkdir -p /mnt/{boot,etc,home}
mount /dev/sda1 /mnt/boot
```
Installing Arch Linux: Install
==============================
```
pacstrap /mnt base
pacstrap /mnt grub-bios
```
Installing and configurating grub
```
genfstab -p /mnt >> /mnt/etc/fstab
arch-chroot /mnt
mkinitcpio -p linux
grub-mkconfig -o /boot/grub/grub.cfg
grub-install /dev/sda
```
Usermanagement
==============
```
useradd -d /home/haraldvdlaan -m -g users -G wheel -s /bin/bash -c "Harald van der Laan" haraldvdlaan
passwd haraldvdlaan
passwd root
systemctl reboot
```
Config changes
==============
```
vi /etc/locale.gen
  uncomment en_US.UTF-8 UTF-8
```
```
locale-gen
localectl set-locale LANG="en_US.UTF-8"
```
Correcting annoying eno16777736 interfaces
```
cat > /etc/udev/rules.d/10-network.rules << EOF
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="aa:bb:cc:dd:ee:ff", NAME="eth0"
EOF
systemctl reboot
```
Activate dhcp
=============
```
systemctl enable dhcpcd@eth0.service
systemctl start dhcpcd@eth0.service
systemctl reboot
```
Installing sudo
===============
```
pacman -S sudo
```
Ensure that users in the group wheel can user sudo
```
vi /etc/sudoers
  uncomment %wheel	ALL = (ALL) ALL
```
Installing xorg
===============
```
pacman -S xorg
pacman -S xterm
pacman -S xorg-xclock
pacman -S xorg-twm
pacman -S xorg-xinit
pacman -S xorg-server-utils
```
Installing gnome desktop manager and gnome
==========================================
```
pacman -S gdm
pacman -S gnome-shell
pacman -S gnome
pacman -S gnome-extras
```
Enable gdm as a service
```
systemctl enable gdm.service
systemctl reboot
```
Installing vmware tools
=======================
```
pacman -S net-tools
pacman -S open-vm-tools open-vm-tools-modules
```
Change KillSignal of vmware tool
```
vi /usr/lib/systemd/system/vmtoolsd.service
  ExecStart=/usr/bin/vmtoolsd
  KillSignal=SIGKILL
```
Enable vmtoolsd as a service
```
systemctl enable vmtoolsd.service
```
Copy arch version to /etc/arch-release
```
cat /proc/version /etc/arch-release
systemctl reboot
```
