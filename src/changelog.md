# Changelog

## Qemu 4 to Qemu 6
 - Based on the initial version by Robert Buhren ([See here](./resources.md))
 - convert Makefile set up to the new ninja based build system
 - integrate with meson
 - add `--enable-gmp` flag to the Qemu configure set up
 - Fix function definitions that changed
 - Create workarounds for deprecated global variables (mostly bios)

## Improvements
 - Added more read only register values to the smn control => Reached another sts output
 - Separated trace output from `LOG_UNIMP` to `LOG_TRACE`

## Status
 - different branch at 0xffff7420 (busy sleep) syncs back up later.
 - different branch at 0xffff3500, syncs back at 0xffff3564
 - different branch at 0xffff35d4, qemu does not reach 0xffff35fc

# TODO
 - YES
