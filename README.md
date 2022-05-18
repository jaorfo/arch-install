# Arch Linux Installation (UEFI/BIOS) and Setup - Xfce Desktop Environment

## Preparation
- Download [Arch Linux ISO](https://archlinux.org/download/)
- Make a booteable pendrive using some software. Like [Balena Etcher](https://www.balena.io/etcher/) 
- Connect the booteable pendrive to your PC and then boot from it

## First steps
- loadkeys es
- ping 8.8.8.8

Show wifi adapter
- ip link
- ip link set wlan0 up
- wpa_passphrase essid password > /etc/wifiConfig (WPA)
- wpa_supplicant -B -i wlan0 -D wext -c /etc/wifiConfig
- dhclient
- ping

Partitions
UEFI
- ls /sys/firmware/efi/efivars
- cfdisk
- if show = gpt
- new 512M -> Type -> EFI System -> Write
- new double RAM -> Type -> Linux swap -> Write
- rest -> Type -> Linux filesystem -> Write

Formatting
- mkfs.vfat -F32 /dev/sda1
- mkfs.ext4 /dev/sda2
- mkswap /dev/sda3
- swapon

Mounting
- mount /dev/sda2 /mnt
- mkdir /mnt/boot
- mkdir /mnt/boot/efi
- mount /dev/sda1 /mnt/boot/efi

Partitions
BIOS
- cfdisk
- if show = dos
- new 512 -> primary -> Booteable -> Type -> 83 Linux -> Write
- new double RAM -> primary -> Type -> 82 Linux swap -> Write
- rest -> primary -> 83 Linux -> Write

Formatting
- mkfs.ext2 /dev/sda1
- mkfs.ext4 /dev/sda2
- mkswap /dev/sda3
- swapon

Mounting
- mount /dev/sda2 /mnt
- mkdir /mnt/boot
- mount /dev/sda1 /mnt/boot

Installing basic packages (UEFI & BIOS)
- pacstrap /mnt linux linux-firmware base nano grub networkmanager dhcpcd netctl wpa_supplicant dialog (efibootmgr)

Generating fstab (UEFI & BIOS)
- genfstab /mnt >> /mnt/etc/fstab
- cat /mnt/etc/fstab (to check)

Configuration (UEFI & BIOS)
- arch-chroot /mnt
- echo YourPCName > /etc/hostname
- ln -sf /usr/share/zoneinfo/America/Argentina /etc/localtime
- nano /etc/locale.gen
- choose: es_AR.UTF8 UTF8
- locale-gen
- hwclock -w
- echo KEYMAP=es > /etc/wconsole.conf
- echo LANG=es_AR.UTF8 > /etc/locale.conf

Installing GRUB (UEFI)
- grub-install --efi-directory=/boot/efi --bootloader -id='Arch Linux' --target=x86_64-efi

Installing GRUB (BIOS)
- grub-install /dev/sda

Configuring GRUB (UEFI & BIOS)
- grub-mkconfig -o /boot/grub/grub.cfg

User and Password (UEFI & BIOS)
root
- passwd
- new password: ...
user
- useradd -m username
- passwd username
- new password: ...
- exit
- umount -R /mnt
- reboot

Wifi Configuration (UEFI & BIOS)
- su
- systemctl start NetworkManager
- systemctl enable NetworkManager
- ip link set wlan up
- nmcli dev wifi connect essid password your_password

Drivers (UEFI & BIOS)
- lspci | grep VGA
- pacman -S xf86-video-nouveau (nvidia) xf86-video-amd (amd) xf86-video-intel intel-ucode (intel) 

Graphical Environment (UEFI & BIOS)
- pacman -S --needed xorg xfce4 xfce4-goodies lightdm-gtk-greeter lightdm-gtk-greeter-settings
- systemctl enable lightdm-gtk-greeter
- reboot 

Others
- sudo pacman -S gvfs git libreoffice-still vlc vim neovim transmission-gtk okular gimp calibre

yay, bluetooth, pulseaudio
