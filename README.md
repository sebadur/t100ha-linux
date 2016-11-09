# Install Linux on Asus T100HA
This is a fully documented how-to install Arch Linux on your very own Asus T100HA. Do not use this for other hardware.
This guide comes *WITHOUT ANY WARRANTY*, use it on your own risk! I am not responsible for any damaged caused.

Still, there are some serious issues with this hardware. Look into https://github.com/sebadur/t100ha-linux/issues for details.

## Use Arch Linux
You will need a *recent* (working scince about version 4.7) Linux kernel and Arch Linux is my recommended option for that. This guide assumes you to use Arch. Otherwise you may be able to skip some the initial steps, but experience some issues not treated with here.
Install it to an USB-thumb-drive:

    dd if=arch.iso of=/dev/sdX status=progress
    sync

## Prepare Installation
Back up your data from Windows, if you need to.
At boot press `[F2]`, then go to `Security -> Secure Boot Menu` and choose the option `Disabled`.
This is needed to enable EFI boot for the up-to-date (unknown signed) kernel. Leave with `[F10] -> [Return]`.
At reboot press `[Esc] -> choose USB-drive`. In the bootloader press `[E]` and append
`nomodeset fbcon=rotate:3` to the vmlinuz starting options. Start with `[Enter]`.

## Android Tethering
You will need internet access in order to install Arch Linux. As the wifi driver will be installed later,
tethering has to be used (or some other internet options):

    # ip link
    # dhcpcd <enp.s..u.u.u.>

## Basic Setup

    (# loadkeys de-latin1)
    # timedatectl set-ntp true
    # fdisk /dev/mmcblk0

Use the following commands within fdisk:

- `d` as often, as needed
- `n`, _default_, _default_, `+512M`
- `t`, `1`
- `n`, _default_, _default_, `+4G`
- `t`, _default_, `19`
- `n`, _default_, _default_, _default_
- `w`

Proceed with:

    # mkfs.fat /dev/mmcblk0p1
    # mkswap /dev/mmcblk0p2
    # mkfs.ext4 /dev/mmcblk0p3
    # swapon /dev/mmcblk0p2
    # mount /dev/mmcblk0p3 /mnt
    # mkdir /mnt/boot
    # mount /dev/mmcblk0p1 /mnt/boot
    # pacstrap /mnt base
    # genfstab -U /mnt >> /mnt/etc/fstab
    # arch-chroot /mnt
    
Replace Region and City to yours:

    # ln -s /usr/share/zoneinfo/Region/City /etc/localtime
    # hwclock --systohc --utc
    # nano /etc/locale.gen
    
Uncomment the locale you need. Save with `[Ctrl]+[O]`, quit with `[Ctrl]+[X]`.

    # locale-gen
    
Use the language you need in the following command:

    # echo LANG=de_DE.UTF-8 > /etc/locale.conf
    (# echo KEYMAP=de-latin1 > /etc/vconsole.conf)
    # echo myhostname > /etc/hostname
    # nano /etc/hosts
    
Insert `127.0.1.1 myhostname.localdomain myhostname`

    # passwd
    # bootctl --path=/boot install
    # blkid -s PARTUUID -o value /dev/mmcblk0p3 > /boot/loader/entries/arch.conf
    # nano /boot/loader/entries/arch.conf

Manipulate the content, so it looks like:

>title          Arch Linux

>linux          /vmlinuz-linux

>initrd         /initramfs-linux.img

>options        root=PARTUUID=........-....-....-....-............ rw video=LVDS-1:d fbcon=rotate:3

    (# pacman -S zsh grml-zsh-config)
    (# chsh -s /bin/zsh)
    
Replace your desired username:

    # useradd -m -G wheel -s /bin/zsh username
    # passwd username
    # pacman -S iasl wget
    # exit
    # umount -R /mnt
    # reboot

## Configure WLAN

    # cat /sys/firmware/acpi/tables/DSDT > dsdt.dat
    # iasl -d dsdt.dat
    # nano dsdt.dsl
    
Search for these two lines and change the marked parts:

>DefinitionBlock ("", "DSDT", 2, "\_ASUS\_", "Notebook", *0x107200A*)

>  ...

>Device (SDHB)

>{

>    Name (*WADR*, Zero)  // \_ADR: Address

    # iasl -tc dsdt.dsl
    # mkdir -p kernel/firmware/acpi
    # cp dsdt.aml kernel/firmware/acpi
    # find kernel | cpio -H newc --create > acpi_override
    # cp acpi_override /boot
    # nano /boot/loader/entries/arch.conf
    
Insert `initrd /acpi_override` (before the boot selector).

    # reboot
    # systemctl enable NetworkManager
    
Check with `ip link` if you got a second internet device. If you don't, execute these commands:

    (# wget https://android.googlesource.com/platform/hardware/broadcom/wlan/+archive/master/bcmdhd/firmware/bcm43341.tar.gz)
    (# tar xf bcm43341.tar.gz)
    (# cp fw_bcm43341.bin /lib/firmware/brcm/brcmfmac43340-sdio.bin)
    (# cp /sys/firmware/efi/efivars/nvram-........-....-....-....-............ /lib/firmware/brcm/brcmfmac43340-sdio.txt)

## Install Desktop Environment
Recommended: Gnome is working fast and well at this netbook.

    # pacman -S gnome gnome-extra
    # systemctl enable gdm
