# System Management Network

The SMN consists of a separate memory region. 32 slots exist to map into the SMN memory region from the PSP memory region.

## PSP memory region

| From | To | Component |
| ---- | -- | ------ |
| `0x01000000` | `0x010fffff` | SMN slot 0 |
| `0x01100000` | `0x011fffff` | SMN slot 1 |
| ...          | ...          | ... |
| `0x02f00000` | `0x02ffffff` | SMN slot 31 |
| ...          | ...          | Not related to SMN |
| `0x03220000` | `0x0322001f` | SMN control |

## SMN memory region
| From | To | Component |
| ---- | -- | ------ |
| ...          | ...          | ... |
| `0x0a000000` | `0x0affffff` | SMN SPI Flash ??? |
| ...          | ...          | ... |
| `0x00000000` | `0xffffffff` | misc (low priority) |

A misc memory mapping covers the whole SMN memory region with low priority.

## Mapping
SMN slot 0 through 31 are mapped into the PSP memory map from address `0x01000000` to `0x02ffffff`.
Every slot can map into a 1MB section of the SMN memory.
The 16 control registers (32 bit each) are located in the PSP memory region at address `0x03220000`.
Each control register corresponds to 2 slot's with the lower and upper 16 bit respectively.
