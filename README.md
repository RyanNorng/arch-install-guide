# arch-install-guide

## Step 1: Create bootable Arch media device
Simple enough, figure it out yourself.

## Step 2: Boot into live environment

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

