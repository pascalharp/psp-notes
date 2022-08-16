# Changelog

## Qemu 4 to Qemu 7
 - Based on the initial version by Robert Buhren ([See here](./resources.md))
 - convert Makefile set up to the new ninja based build system
 - integrate with meson
 - add `--enable-gmp` flag to the Qemu configure set up
 - Fix function definitions that changed
 - Create workarounds for deprecated global variables (mostly bios)
 - merged upstream to upgrade to Qemu 7 and fixed deprecated stuff

## Sync Bridge
 - Created Sync Bridge Plugin
 - Sync Bridge Plugin now compares most basic registers instead of only pc
 - Moved Sync Bridge to its own repository

## Improvements
 - Added more read only register values to the smn control => Reached another sts output
 - Separated trace output from `LOG_UNIMP` to `LOG_TRACE`
 - Separated psp fuse to own dev device
 - Add debug option do fuse device (to match PSPEmu behavior)
 - Fix reset sctlr

## Status
 - different branch at 0xffff7420 (busy sleep) syncs back later.
 - ~~registers diverge at 0xffff0b60 -> value loaded from smn reagion differs (addr: 0x125a078)~~
 - registers diverge at 0xffff3f14 -> Different read from timer -> syncs back later
 - registers diverge at 0xffff40f8 -> Different read from timer -> syncs back later
 - registers diverge at 0xffff63f8 -> Unimplemented device in qemu -> Results in different branch after 0xffff640c
   - Qemu to: 0xffff6410
   - PSPEmu to: 0xffff6428
   - PC syncs back up at 0xffff6428, reg values still differ, results in pc difference after 0xffff62a8
