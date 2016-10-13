# Install Linux on Asus T100HA
This is a fully documented how-to install Arch Linux on your very own Asus T100HA. Do not use this for other hardware.
This guide comes *WITHOUT ANY WARRANTY*, use it on your own risk! I am not responsible for any damaged caused.

## Use Arch Linux
You will need the *most* recent Linux kernel and Arch Linux is my recommended option for that.
Install it to an USB-thumb-drive:

    dd if=arch.iso of=/dev/sdX status=progress
    sync

## Prepare Installation
Back up your data from Windows, if you need to.
At boot press `[F2]`, then go to `Security -> Secure Boot Menu` and choose the option `Disabled`.
This is needed to enable EFI boot for the up-to-date (unknown signed) kernel. Leave with `[F10] -> [Return]`.
At reboot press `[Esc] -> choose USB-drive`. In the bootloader press `[E]` and append
`nomodeset fbcon=rotate:3` to the vmlinuz starting options. Start with `F10`.

## Android Tethering
You will need internet access in order to install Arch Linux. As the wifi driver will be installed later,
tethering has to be used (or some other internet options):

    # ip link
    # dhcpcd <enp.s..u.u.u.>

## Basic Setup

    (# loadkeys de-latin1)
    # timedatectl set-ntp true
Use fdisk to format your partitions as follows: `/dev/mmcblk0`: _p1_: 512M EFI, _p2_: 53.8G ext4, _p3_: 4G swap

    # mkfs.ext4 /dev/mmcblk0p2
    # swapon /dev/mmcblk0p3
    # mount /dev/mmcblk0p2 /mnt
    # mkdir /mnt/boot
    # mount /dev/mmcblk0p1 /mnt/boot
    # pacstrap /mnt base
    # genfstab -U /mnt >> /mnt/etc/fstab
    
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
    # nano /boot/loader/loader.conf

>default  ...............-*

>timeout  4

>editor   0

    # blkid -s PARTUUID -o value /dev/mmcblk0p2 > /boot/loader/entries/arch.conf
    # nano /boot/loader/entries/arch.conf

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

## Issues
Still, there are some issues:

- Sleep and Suspend (S1 to S3) not working properly. This is a problem, because every time you close the netbook lid it sends the machine to suspend. This should be turned off, until a solution is found. It is for sure another wrong configuration in the ACPI DSDT. By the way: It may help to update the EFI version firmware, if you can't get wifi working, just don't forget to repeat the WLAN-configuration steps again.

- Background Brightness fixed at 100%. There are people claiming solutions for that, that have to be verified first.

- Sound: Not working this way, but also called fixable.

- Bluetooth: Not tested.

- Cameras: Not working.
