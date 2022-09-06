# Sync Bridge
Sync Bridge is a small little python script to synchronize two running gdb instances. It is used to compare the behavior of PSPEmu and the Qemu port. It currently only compares the PC and stops once they differ.

## python script
The script can be found in this [repository](https://github.com/pascalharp/gdb_sync_bridge)

## Usage

First start the Qemu and the PSPEmu emulation with gdb. Different ports are required. Here PSPEmu will be on port `1235` and Qemu will be on the default port `1234`.

Replace `PSP_ROM_BL` with the full path to the on-chip bootloader and `PSP_UEFI_IMAGE` with the full path to the uefi image.

Now start two `arm-none-eabi-gdb` instances and connect one to the Qemu emulation and one to the PSPEmu emulation. Source the sync\_bridge plugin and initialize it. Now first start one of them first as the leader with `sb_leader` and afterwords the other one as the follower `sb_follow`. Make sure they start at the same address. Check the scripts below on how to automate this process.

The registers to compare are exchanged before the synchronization starts. Additionally address sections can be specified with `--skip start:end` that should be skipped and not checked.
Example:
```
sb_lead --skip 0xffff7418:0xffff06dc --skip 0xffff3f14:0xffff3f80 --skip 0xffff40f8:0xffff4128 --skip 0xffff7978:0xffff798c
```

### Qemu emulation
```bash
#!/bin/env bash
path/to/qemu/build/qemu-system-arm \
	--singlestep \
	--machine amd-psp_zen \
	--display none \
	-device loader,file=$PSP_ROM_BL,addr=0xffff0000,force-raw=on \
	-global amd-psp.dbg_mode=true \
	-bios $PSP_UEFI_IMAGE \
	-serial stdio \
	-d unimp,guest_errors
	-S -s
```

### PSPEmu emulation
```bash
#!/bin/env bash
path/to/PSPEmu/build/PSPEmu \
    --emulation-mode on-chip-bl \
    --flash-rom $PSP_UEFI_IMAGE \
    --on-chip-bl $PSP_ROM_BL \
    --timer-real-time \
    --trace-log ./log \
    --intercept-svc-6 \
    --trace-svcs \
    --dbg 1235
```

### GDB qemu
```bash
#!/bin/env bash
arm-none-eabi-gdb \
	-ex 'source path/to/ret-sync/sync.py' \  # Only if combined with ret-sync
	-ex 'ser arch arm' \
	-ex 'source path/to/sync_bridge.py' \    # Set path to sync bridge script
	-ex 'sb_init' \
	-ex 'target remote localhost:1234' \
	-ex 'break *0xffff005c' \                # Set this to the starting point
	-ex 'sync' \                             # Only if combined with ret-sync
	-ex 'continue'
```

Then initiate the synchronization with `sb_lead`

### GDB PSPEmu
```bash
#!/bin/env bash
arm-none-eabi-gdb \
	-ex 'ser arch arm' \
	-ex 'source path/to/sync_bridge.py' \    # Set path to sync bridge script
	-ex 'sb_init' \
	-ex 'target remote localhost:1235' \
	-ex 'break *0xffff005c' \                # Set this to the starting point
	-ex 'continue'
```

Let the synchronization begin with `sb_follow`
