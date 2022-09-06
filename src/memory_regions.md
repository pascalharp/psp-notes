# memory regions

| From | To | Periph |
| ---- | -- | ------ |
| `0x00000000` | `0x00040000` | SRAM |
| `0x01000000` | `0x02ffffff` | [System Management Network Mapped](./memory_regions/smn.md) |
| `0x03000000` | `0x03ffffff` | [Standard MMIO](./memory_regions/mmio.md) |
| `0x04000000` | `0xfbffffff` | x86 address space mapping slots |
| `0xffff0000` | `0xffffffff` | ROM (on-chip boot loader) |
