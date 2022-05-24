# Comparison PSPEmu vs Qemu

## General MMIO

| What        || PSPEmu | Where                || Qemu  | Where                    |
| ----        || ------ | -----                || ----  | ------------------------ |
| psp timer   || ✅     | `psp-dev-timer.c`    || ✅    | `hw/arm/psp-timer.c`     |
| psp sts     || ✅     | `psp-dev-status.c`   || ✅    | `hw/arm/psp-sts.c`       |
| psp irq     || ✅     | `psp-irq.c`          || ❌    |                          |
| smn control || ✅     | ?                    || ✅    | `hw/arm/psp-smn.c`       |
| x86 control || ✅     | ?                    || ❌ \* | `hw/arm/psp-x86.c`       |
| CCP         || ✅     | `psp-dev-ccp-v5.c`   || ✅    | `hw/misc/ccpv5.c`        |
| smn flash   || ✅     | `psp-dev-flash.c`    || ✅    | `hw/arm/psp-smn-flash.c` |
| psp fuse    || ✅     | `psp-dev-fuse.c`     || ❌    |                          |
| psp gpio    || ✅     | `psp-dev-gpio.c`     || ❌ ?  |                          |
| x86 uart    || ✅     | `psp-dev-x86-uart.c` || ❌ ?  |                          |
| x86 iomux   || ✅     | `psp-dev-iomux.c`    || ❌ ?  |                          |
| x86 lpc     || ✅     | `psp-dev-lpc.c`      || ❌ ?  |                          |

_* only on Qemu v4 at the moment_

## Misc
 - PSPEmu has a svc implementation?
 - PSPEmu smu?

## SMN
TODO

## x86
TODO
