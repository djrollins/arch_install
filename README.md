# Arch installation notes

Theses are my own personal notes for my preferred setup for Arch Linux.

This primarily exists to remind me how to install Arch in the future, but I am
publishing them should they potentially help someone else.

# Preparation

1) load console keymap
```
$ ls /usr/share/kbd/keymaps/**/*.map.gz
$ loadkeys uk # most likely.
```

2) synchronize clock
```
$ timedatectl set-ntp true
```

3) partition disks if using EFI (EFY must be formatted as FAT32).
 - `fdisk` if dual booting with windows
 - `gdisk` for gtp paritional tables
 - `c{f,g}disk` for curses interface

4) Make btrfs file systems

Creating a btrfs filesystem directly on the raw device doesn't allow for EFI,
create an EFI partition if using EFI.

    1) `mkfs.btrfs -L "Arch Linux" /dev/sdXX`
    2) `mount /dev/sdXX /mnt/btrfs-{home,root}`
    3) `__current` and `__snapshot` directories for each filesystem
    4) `btrfs subvolume create ...`
        1) `/mnt/btrfs-home/__current/@home`
        2) `/mnt/btrfs-root/__current/@`
        3) `/mnt/btrfs-root/__current/@boot
    5) `mount -o ...,subvol=__current/@  /dev/sdXX /mnt/btrfs_current`
    7) `mount -o ...,subvol=__current/@boot /dev/sdXX /mnt/btrfs_current/boot`
    8) `mount -o ... /dev/sdXX /mnt/btrfs_current/boot/efi`
    6) `mount -o ...,subvol=__current/@home /dev/sdXX /mnt/btrfs_current/home`
    7) consider other options `-o compress=lzo,discard,ssd,relatime` etc

# Installation

5) Install arch with `pacstrap /mnt/btrfs-current base base-devel`

6) Generate fstab w/ UUIDs and clean them up
    1) `genfstab -U /mnt/btrfs-current >> /mnt/btrfs-current/etc/fstab`
    2) clean up extraneous options that genfstab generates e.g. `subvolid=` and multiple `subvol=` etc.
    3) for each UUID mount them the `/run/btrfs-root` or `/run/btrfs-home` etc for easy snapshotting

# Configuration

7) `arch-chroot /mnt/btrfs-current`

8) set the hardware clock from system clock `hwclock --systohc --utc`

9) edit `/etc/mkinicpio.conf` and generate initial ramdisk
    1) edit add `btrfs` to `HOOKS` and install `btrfs-progs`
    2) generate the ramdisk `mkinicpio -p linux`

10) install bootloader and utilities to find windows partition
    1) `pacman -S grub os-prober ntfs-3g` 
    2) `grub-install /dev/sdX`
    3) `grub-mkconfig -o /boot/grub/grub.cfg`

11) reboot

# post install

12) Set root password `passwd`

13) `timedatectl set-timezone Europe/London`
    13.1) or `ln -sf usr/share/zoneinfo/Europe/London /etc/localtime`

14) set and generate locales
    1) uncomment `en_GB` in `/etc/locale.gen`
    2) generate locales `locale-gen`
    3) `localectl set-locale LANG=en_GB.UTF-8`
        3.1) or `echo LANG=en_GB.UTF-8 > /etc/locale.conf`
    4) `localectl set-keymap uk`
        4.1) or `echo KEYMAP=uk > /etc/vconsole.conf`
    5) `localectl set-x11-keymap gb pc105 "" caps:nocaps`
    
15) `hostnamectl set-hostname <hostname>`
    15.1) or `echo <hostname> > /etc/hostname`

16) Add users
    1) `useradd -m -G wheel -c "Daniel J. Rollins" djr`
    2) `passwd djr`

17) Add wheel group to sudo.
    1) `visudo`
    2) Uncomment `%wheel ALL=(ALL) ALL`

18) Enable dhcpcd for all network adapters
    1) systemctl enable --now dhcpcd.service
    
19) install `nvidia` or `nvidia-dkms`
    1) add `nvidia-drm.modeset=1 video=vesafb:mtrr:3,ywrap` to kernel parameters in `/etc/defaults/grub` to enable KMS and speed up vconsole respectively
    2) also consider `GRUB_TERMINAL_OUTPUT=console` if grub renders slowly.
    3) regenerate grub config after the above steps.
    4) add `nvidia nvidia_drm nvidia_uvm nvidia_modeset` to mkinitcpio.conf so the the above works on boot

# Install DE

TODO
