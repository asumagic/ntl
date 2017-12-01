---
title: Memory and registers
---

Memory and registers
=====

The 16 registers are __16-bit__ wide. Memory addressing is __16-bit__, and the memory atom type is __16-bit__, effectively enabling 128KiB memory.  
When the machine is initialized, every register defaults to `0`.

ntl is a __load-store__ architecture - all memory accesses are performed through `load` and `store`.  
It is a __von Neumann__ architecture, so the data and program memory are the same.

#### Register cheatsheet

| Name   | ID    | Description           |
|--------|-------|-----------------------|
| `rfl`  | `0x0` | Flag register         |
| `ridt` | `0x1` | IDT address           |
| `racc` | `0x2` | Accumulator           |
| `rsp`  | `0x3` | Stack pointer         |
| `r4`   | `0x4` | General purpose #4    |
| `r5`   | `0x5` | General purpose #5    |
| `r6`   | `0x6` | General purpose #6    |
| `r7`   | `0x7` | General purpose #7    |

### Flag register

The `fl` register is the flag register, which stores some useful flags.

#### Flag register cheatsheet

| Name       | Offset | Description                        |
|------------|--------|------------------------------------|
| `_INTON`   | `0x0`  | Interrupts globally enabled        |
| `_INT1ON`  | `0x1`  | Interrupt #1 enabled               |
| `_INT2ON`  | `0x2`  | Interrupt #2 enabled               |
| `_INT3ON`  | `0x3`  | Interrupt #3 enabled               |
| `_INT4ON`  | `0x4`  | Interrupt #4 enabled               |
| `_INT5ON`  | `0x5`  | Interrupt #5 enabled               |
| `_INT6ON`  | `0x6`  | Interrupt #6 enabled               |
| `_INT7ON`  | `0x7`  | Interrupt #7 enabled               |
| `_TEST`    | `0x8`  | Test was positive                  |
| `_INTLOCK` | `0xA`  | Interrupt lock                     |
