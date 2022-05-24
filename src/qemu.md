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

## Connect with gdb

Either with `arm-none-eabi-gdb` or `gdb-multiarch` you can connect to the running Qemu session with `target remote localhost:1234` if the emulation was started with the `-s` flag
