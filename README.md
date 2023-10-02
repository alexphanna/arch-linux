# Arch-Linux
[Arch Linux](https://archlinux.org/) installation guide
## Pre-installation
### Partition the disks
```sh
fdisk -l
fdisk /dev/the_disk_to_be_partitioned
```
### Format the partitions
```sh
mkfs.ext4 /dev/root_partition
mkfs.fat -F 32 /dev/efi_system_partition
```
### Mount the file systems
```sh
mount /dev/root_partition /mnt
mount --mkdir /dev/efi_system_partition /mnt/boot
```
### Swap file creation
```sh
dd if=/dev/zero of=/swapfile bs=1M count=8k status=progress
chmod 0600 /swapfile
mkswap -U clear /swapfile
swapon /swapfile
echo "/swapfile none swap defaults 0 0" >> /etc/fstab
```
## Installation
### Install essential packages
```sh
pacstrap -K /mnt amd-ucode base base-devel linux linux-firmware
```
## Configure the system
### Fstab
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```
### Chroot
```sh
arch-chroot /mnt
```
### Time zone
```sh
ln -sf /usr/share/zoneinfo/US/Eastern /etc/localtime
hwclock --systohc
```
### Localization
Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8`
```sh
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```
### Network configuration
```sh
echo "myhostname" > /etc/hostname
echo -e "[Match]\nName=en*\n\n[Network]\nDHCP=yes" > /etc/systemd/network/20-wired.network
sudo systemctl enable --now systemd-networkd.service
sudo systemctl enable --now systemd-resolved.service
```
### Root password
```sh
passwd
```
### Boot loader: systemd-boot
```sh
bootctl install
echo -e "default arch.conf\ntimeout 4\nconsole-mode max\neditor no" > /boot/loader/loader.conf
echo -e "title Arch Linux\nlinux /vmlinuz-linux\ninitrd /amd-ucode.img\ninitrd /initramfs-linux.img\noptions root="LABEL=Arch OS" rw" > /boot/loader/entries/arch.conf
```
### System administration
Uncomment `%wheel      ALL=(ALL:ALL) ALL` in `visudo`
```sh
useradd -m -G wheel alex
passwd alex
```
### pacman
Uncomment the `[multilib]` section in `/etc/pacman.conf`

Edit `ParallelDownloads` under `[options]` to a positive integer in `/etc/pacman.conf`
## Post-installation
### Graphical user interface
#### Display server : Xorg
```sh
sudo pacman -S xorg-server xorg-xinit xorg-randr
xrandr --output HDMI-0 --mode 1920x1080 --rate 74.97
```
#### Display drivers : Nvidia
```sh
sudo pacman -S nvidia nvidia-utils lib32-nvidia-utils
```
#### Window managers : dwm
```sh
sudo pacman -S freetype2 git libx11 libxft ttc-iosevka
mkdir -p ~/.local/src
git clone https://git.suckless.org/dwm ~/.local/src/dwm
cd ~/.local/src/dwm
```
Comment `Xinerma` section in `config.mk` 
```sh
make clean
sudo make install
echo "exec dwm" > ~/.xinitrc
```
