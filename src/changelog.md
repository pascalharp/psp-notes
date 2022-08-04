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

## Status
 - different branch at 0xffff7420 (busy sleep) syncs back up later.
 - different branch at 0xffff3500, syncs back at 0xffff3564
 - different branch at 0xffff35d4, qemu does not reach 0xffff35fc

## Notes
 - good sync at 0xffff06dc
 - entering func 0xffff340c
   - psp-qemu has different value in r1 (0x0 instead of 0x10)
   - jumped to from 0xffff0cac
   - r1 copied from r9
