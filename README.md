# arch-install-guide

## Step 1: Create bootable Arch media device
Simple enough, figure it out yourself.

## Step 2: Boot into live environment

### Pre-partition setup
1.1 Set the console keybooard layout (US by default):
- list available keymaps with `localectl list-keymaps`; and,
- load the keymap with `loadkeys <your keymap here>`.
  
1.2 Verify the boot mode
  `cat /sys/firmware/efi/fw_platform_size`
  - If the command returns 64, the system is booted in UEFI mode and has a 64-bit x64 UEFI.
  - If the command returns 32, the system is booted in UEFI mode and has a 32-bit IA32 UEFI. While this is supported, it will limit the boot loader choice to those that support mixed mode booting.
  - If it returns No such file or directory, the system may be booted in BIOS (or CSM) mode.
If the system did not boot in the mode you desired (UEFI vs BIOS), refer to your motherboard's manual.

1.3 Connect to the internet
- Use the `iwctl` utility for this purpose; and,
- Confirm that your connection is active with `ping -c 2 archlinux.org`.

1.4 Update system clock
- Use `timedatectl`
- `timedatectl set-ntp true` if clock is not synchronized

### Partition the disks

2.1 `lsblk` to list devices

2.2 `cfdisk /dev/the_disk_to_be_partitioned` to partition the disk (use the -z flag to zero out disk if it's already partitioned)
- efi boot partition should be 1GiB
- swap should be at least 4GiB (but for my purposes, I will set it equal to my ram (32G)
- rest will go to root partition

### Format the partitions

3.1 `mkfs.btrfs /dev/root_partition`

3.2 `mkswap  /dev/swap_partition`

3.3 `mkfs.fat -F 32 /dev/efi_system_partition`

### Disk Mounting

4.1 `swapon /dev/swap_partition` to enable swap

4.2 `mkdir -p /mnt/efi`

4.3 `mount /dev/efi_system_partition /mnt/efi`

In general there are 2 main mountpoints to use: /efi or /boot but in this configuration i am forced to use /efi, because by choosing /boot we could experience a system crash when trying to restore @ ( the root subvolume ) to a previous state after kernel updates. This happens because /boot files such as the kernel won't reside on @ but on the efi partition and hence they can't be saved when snapshotting @. Also this choice grants separation of concerns and also is good if one wants to encrypt /boot, since you can't encrypt efi files. 

### BTRFS Subvolume setup

4.1 mount the root fs

- `mount /dev/root_partition /mnt`
  
4.2 create the subvolumes

`btrfs su cr /mnt/@`

`btrfs su cr /mnt/@home`

`btrfs su cr /mnt/@cache`

`btrfs su cr /mnt/@log`


4.3 unmount the root fs

`umount /mnt`

4.4 compress the btrfs subvolumes with Zstd

`mount -o subvol=/@,defaults,noatime,compress=zstd /dev/root_partition /mnt`

`mount -o subvol=/@home,defaults,noatime,compress=zstd -m /dev/root_partition /mnt/home`

`mount -o subvol=/@cache,defaults,noatime,compress=zstd -m /dev/root_partition /mnt/var/cache`

`mount -o subvol=/@log,defaults,noatime,compress=zstd -m /dev/root_partition /mnt/var/log`

## Step 3: Installation

### Set up mirrors

`reflector --country US --latest 5 --sort rate --save /etc/pacman.d/mirrorlist`

### Pacstrap

- "base, linux, and linux-firmware" are mandatory packages
- "base-devel" for base developer packages
- "git" for git
- "btrfs-progs" is a user-space utility for btrfs file-system management
- "grub" the bootloader
- "efibootmgr" needed to install grub
- "grub-btrfs" btrfs support for grub
- "vi" required to visudo and give myself sudo privileges 
- "networkmanager" to connect to the internet
- "amd-ucode" for amd microcode
- "sudo" to allow sudo

`pacstrap -K /mnt base linux linux-firmware base-devel git btrfs-progs grub efibootmgr grub-btrfs vi networkmanager amd-ucode sudo`

## Step 4: Configure the system

### Generate fstab

- `genfstab -U /mnt >> /mnt/etc/fstab`
- `cat /mnt/etc/fstab` to see the result

### Chroot into the system

- `arch-chroot /mnt`

### Time

- list timezones with `timedatectl list-timezones`
- `ln sf /usr/share/zoneinfo/US/Pacific /etc/localtime`
- `hwclock --systohc`

### Localization

- `vim /etc/local.gen` and uncomment en_US.UTF -8 and en_US ISO-8856-1
- `locale-gen` to generate the locale
- `vim /etc/locale.conf` and let the language variable to `LANG=en_US.UTF-8`

### mkinitcpio config

- with btrfs, I'll need to edit /etc/mkinitcpio.conf before regenerating it
- `vim /etc/mkinitcpio.conf` and set MODULES=(btrfs)
- regenerate the initramfs with `mkinitcpio -P`

### Create the hostname file

`echo "ArchDesktop" >> /etc/hostname`

### Set the root password and add users

- `passwd`
- `useradd -m -G wheel,users norng`
- `passwd norng`
- give myself sudo privleges by using the command `visudo` and deleting the comment for the wheel group allowing users in the wheel group sudo privileges




### Install bootloader







