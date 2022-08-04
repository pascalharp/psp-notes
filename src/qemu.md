# Qemu

The PSPEmu port for Qemu can be found [here](https://github.com/pascalharp/qemu/tree/psp_refactor)

## Building Qemu

 - Clone the repository and check out the correct branch
```bash
git clone git@github.com:pascalharp/qemu.git
cd qemu
git checkout psp_refactor
```
 - Create a build directory and run the configure script
 ```bash
mkdir build
cd build
../configure --target-list=arm-softmmu --enable-debug --disable-tools --disable-guest-agent --disable-virtfs --enable-nettle --enable-gmp
 ```

 - Build Qemu
 ```bash
 make
 ```

 The final built of `qemu-system-arm` will be located inside the build directory.

## Run the emulation
Run the emulation (assuming you are inside the build directory) with:
```bash
./arm-softmmu/qemu-system-arm --singlestep --machine amd-psp_zen --display none -device loader,file=$PATH_TO_ROM_BL,addr=0xffff0000,force-raw=on -bios $PATH_TO_UEFI_IMAGE -serial stdio
```
Replace `$PATH_TO_ROM_BL` with the path to the on-chip boot loader image and `PATH_TO_UEFI_IMAGE` with the path to the full uefi image. The trace output is emitted on stderr.

Additionally you can use:

- `-d unimp,guest_errors` to output additional information (very verbose!).
- `-S -s` to stop the execution at the beginning and to open a gdb port.

### Additional emulation options
Set specific device properties with the `-global` option. For example: `-global amd-psp.dbg_mode=true`

Options:
 - `amd-psp.dbg_mode` bool, enable dbg mode. This corresponds to the `fPspDbgMode` flag on PSPEmu.

## Connect with gdb

Either with `arm-none-eabi-gdb` or `gdb-multiarch` you can connect to the running Qemu session with `target remote localhost:1234` if the emulation was started with the `-s` flag

## ret-sync
ret-sync can be used to synchronize a running gdb session with ghidra.

### ghdira
Download the latest [ret-sync extension for ghidra](https://github.com/bootleg/ret-sync/tree/master/ext_ghidra/dist) and the matching [ghidra version](https://github.com/NationalSecurityAgency/ghidra/releases). As of now ghidra does not have a stable plugin api, so the version has to match exactly. Create a new ghidra project (and probably import the already existing ghidra project) and install the extension: `File -> Install Extension -> + -> ...`. Inside the directory where the ghidra project is located create a `.sync` configuration file:
```bash
cat <<EOF > .sync
    [GENERAL]
    use_raw_addr=true
EOF
```
If everything was set up correctly you should see output similar to this after restarting ghidra and opening the project:
```bash
[*] retsync init
[>] loading configuration file /path/to/project/psp.rep/.sync
  - using raw addresses: true
```
In the top menu you should now see new options for ret-sync. Click the `+` to Start the listener.

### gdb
A gdb version for arm compiled with python3 support is required. Download the [ret-sync plugin for gdb](https://github.com/bootleg/ret-sync/tree/master/ext_gdb) and save it. In the same directory create a `.sync` configuration file with the corresponding mapping. For example:
```bash
cat <<EOF > .sync
    [INIT]
    context = {
        "pid": 200,
        "mappings": [
          [0x0, 0x1ffff, 0x20000, "OFF_CHIP_BL"],
          [0x00e00000, 0x00efffff, 0x100000, "ASRock-A520M-HSV-1.31.fd90.driver_entries.bin"],
          [0xffff0000, 0xffffffff, 0x10000, "on-chip-ryzen-zen.bl"]
        ]
      }
EOF
```
Start the Qemu emulation with debug flags (see above). Then start gdb, attach to the session, source the ret-sync plugin and start the synchronization:
```
arm-none-eabi-gdb
target remote localhost:1234
source /path/to/plugin/sync.py
sync
```
The output should look similar to this:
```
(gdb) target remote localhost:1234
Remote debugging using localhost:1234
warning: No executable has been specified and target does not support
determining executable automatically.  Try using the "file" command.
0xffff0000 in ?? ()
(gdb) source /path/to/plugin/sync.py
[sync] configuration file loaded from: /path/to/plugin/.sync
[sync] initialization context:
{
    "pid": 200,
    "mappings": [
        [
            0,
            131071,
            131072,
            "OFF_CHIP_BL"
        ],
        [
            14680064,
            15728639,
            1048576,
            "ASRock-A520M-HSV-1.31.fd90.driver_entries.bin"
        ],
        [
            4294901760,
            4294967295,
            65536,
            "on-chip-ryzen-zen.bl"
        ]
    ]
}

[sync] init
[sync] 18 commands added
(gdb) sync
[sync] initializing tunnel to IDA using localhost:9100...
[sync] sync is now enabled with host localhost
(gdb)
```

You can now step through the code in gdb and ghidra will update or set [breakpoints and continue the emulation](https://github.com/bootleg/ret-sync#ida-bindings-over-debugger-commands) from within ghidra.
