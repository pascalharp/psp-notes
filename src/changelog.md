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
 - Use python calls to load register values instead of gdb cli calls
 - Automatically exchange compatible registers for synchronization
 - Add option to skip address ranges to be checked

## Improvements
 - Added more read only register values to the smn control => Reached another sts output
 - Separated trace output from `LOG_UNIMP` to `LOG_TRACE`
 - Separated psp fuse to own dev device
 - Add debug option do fuse device (to match PSPEmu behavior)
 - Fix reset sctlr
 - Fix smn misc values
 - Add multiple machines to support Zen+ and Zen2
 - Create different class implementations for soc and smn to support different mapping configurations
 - Rework tracing to support qemu's `-d` flag, check `qemu-system-arm -d "trace:help" | grep -E 'psp|ccp'` for list

## Status
 - Zen and Zen+ on-chip bootloader working
 - Zen2 stuck half way
