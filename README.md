<img src="https://github.com/orfoja/img/blob/main/archlogo.png?raw=true" alt="archlinux" style="width:200px;"/>

# Arch Linux Installation (UEFI/BIOS) and Setup - Xfce Desktop Environment

## Preparation
- Download [Arch Linux ISO](https://archlinux.org/download/)
- Make a booteable pendrive using some software. Like [Balena Etcher](https://www.balena.io/etcher/) 
- Connect the booteable pendrive to your PC and then boot from it

## First steps
- load keyword language (example spanish)
```console
root@archiso# loadkeys es
```

## Wifi Configuration
- Get the wifi adapter name (wlan0 in this example)
```console
root@archiso# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlan0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DORMANT group default qlen 1000
    link/ether g2:gb:6g:6g:g3:g9 brd ff:ff:ff:ff:ff:ff permaddr f4:fg:gd:d6:97:gg
```
- Set the wifi adapter up
```console
root@archiso# ip link set wlan0 up
```
- Configure wifi
```console
root@archiso# wpa_passphrase essid password > /etc/wifiConfig
root@archiso# wpa_supplicant -B -i wlan0 -D wext -c /etc/wifiConfig
root@archiso# dhclient
```
- Test connection
```console
root@archiso# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=10.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=13.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=116 time=11.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=116 time=11.9 ms
...
```
# IMPORTANT! 
## Check if your computer has UEFI or BIOS
- This is the UEFI output look like
```console
root@archiso# ls /sys/firmware/efi/efivars
 AcpiGlobalVariable-af9ffd67-ec10-488a-9dfc-6cbf5ee22c2e
 AcpiGlobalVariable-c020489e-6db2-4ef2-9aa5-ca06fc11d36a
 AdvMitAttrib-ec87d643-eba4-4bb5-a1e5-3f3e36b20da9
 AfterReadyToBoot-7b77fb8b-1e0d-4d7e-953f-3980a261e077
 AmiGopPolicySetupData-ec87d643-eba4-4bb5-a1e5-3f3e36b20da9
 AMITSESetup-c811fa38-42c8-4579-a9bb-60e94eddfb34
 BiosSetupType-ec87d643-eba4-4bb5-a1e5-3f3e36b20da9
 ...
```
- This is the BIOS output look like
```console
root@archiso# ls /sys/firmware/efi/efivars
file or directory does not exist
```
### Summing up: UEFI = some output, BIOS = directory not found


## Partitions (UEFI)
- Run to enter the partition manager (if a menu shows up, choose GPT option)
```console
root@archiso# cfdisk
```
- Basic partitions

| Partition | Size              | Id Type          |
| --------- | ----------------- | ---------------- |
| BOOT      | 512M              | EFI System       |
| SWAP      | Double RAM        | Linux swap       | 
| SYSTEM    | Rest of your GB   | Linux filesystem |

## Format (UEFI)
```console
root@archiso# mkfs.vfat -F32 /dev/sda1
root@archiso# mkfs.ext4 /dev/sda2
root@archiso# mkswap /dev/sda3
root@archiso# swapon
```

## Mount (UEFI)
```console
root@archiso# /dev/sda2 /mnt
root@archiso# /mnt/boot
root@archiso# /mnt/boot/efi
root@archiso# /dev/sda1 /mnt/boot/efi
```

## Partitions (BIOS)
- Run to enter the partition manager (if a menu shows up, choose DOS option)
```console
root@archiso# cfdisk
```
- Basic partitions

| Partition | Type     | Size              | Id Type          |
| --------- | -------- | ----------------- | ---------------- |
| BOOTEABLE | Primary  | 512M              | 83 Linux         |
| SWAP      | Primary  | Double RAM        | 82 Linux swap    | 
| SYSTEM    | Primary  | Rest of your GB   | 83 Linux         |

## Format (BIOS)
```console
root@archiso# mkfs.ext2 /dev/sda1
root@archiso# mkfs.ext4 /dev/sda2
root@archiso# mkswap /dev/sda3
root@archiso# swapon
```

## Mount (BIOS)
```console
root@archiso# /dev/sda2 /mnt
root@archiso# /mnt/boot
root@archiso# /dev/sda1 /mnt/boot
```

## Install basic packages (UEFI)
```console
root@archiso# pacstrap /mnt linux linux-firmware base nano grub networkmanager dhcpcd netctl wpa_supplicant dialog efibootmgr
```

## Install basic packages (BIOS)
```console
root@archiso# pacstrap /mnt linux linux-firmware base nano grub networkmanager dhcpcd netctl wpa_supplicant dialog
```

## Generate fstab (UEFI & BIOS)
```console
root@archiso# genfstab /mnt >> /mnt/etc/fstab
```
- Check 
```console
root@archiso# cat /mnt/etc/fstab
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# UUID=XXXXX-XXXX-XXXX-XXXX-XXXX
/dev/sda2           	/         	ext4      	rw,relatime	0 1
...
```

## Host, Clock and Locale configurations (UEFI & BIOS)
- Mount the filesystem
```console
root@archiso# arch-chroot /mnt
```
- Set Host name (whatever name you choose)
```console
root@archiso# echo your_host_name > /etc/hostname
```
- Set localtime (choose yours, this is an example with Argentina)
```console
root@archiso# ln -sf /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime
```
- Set keyword distribution
```console
root@archiso# nano /etc/locale.gen
```
- Uncomment the distribution you want (in this example: es_AR.UTF8 UTF8)
<img src="https://github.com/orfoja/img/blob/main/arch-locale.png?raw=true" alt="archlinux" style="width:600px;"/>

- Save file pressing `Ctrl+O` and `Ctrl+X` to exit
- Execute to generate locale:
```console
root@archiso# locale-gen
Generating locales...
    es_AR.UTF8... done
Generation complete.
```
- Set Clock
```console
root@archiso# hwclock -w
```
- Set language (again, this is an example with Argentina)
```console
root@archiso# echo KEYMAP=es > /etc/wconsole.conf
root@archiso# echo LANG=es_AR.UTF8 > /etc/locale.conf
```

## Installing GRUB (UEFI)
- Run `grub-install` with this params
```console
root@archiso# grub-install --efi-directory=/boot/efi --bootloader -id='Arch Linux' --target=x86_64-efi
Installing for x86_64-efi platform
Installation finished. No error reported.
```
- Configure grub
```console
root@archiso# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub installation file...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot: initramfs-linux-fallback.img
Warning: os-prober will not be executed to detect other booteable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

## Installing GRUB (BIOS)
- Run `grub-install` with this params
```console
root@archiso# grub-install /dev/sda
Installing for i386-pc platform
Installation finished. No error reported.
```
- Configure grub
```console
root@archiso# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub installation file...
Found linux image: /boot/vmlinuz-linux
Found initrd image: /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot: initramfs-linux-fallback.img
Warning: os-prober will not be executed to detect other booteable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for ...
done
```

## User and Password (UEFI & BIOS)
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
