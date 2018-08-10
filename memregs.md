---
title: Memory and registers
---

Memory and registers
=====

All 16 registers are __16-bit__ wide. Memory addressing is __16-bit__, and the memory atom type is __8-bit__, effectively enabling a 64KiB address space.

When the machine is initialized, registers are in an uninitialized state, with the following exceptions:

- `rip` is initialized by an implementation-defined value, so the machine boots into its firmware successfully.
- `rfl` is initialized to zero.

The main memory hosts the program and data within the same address space and can be manipulated through specialized instructions only.  
See [Instructions](instructions.md)

### Register cheatsheet

| Name   | ID    | Description                               |
|--------|-------|-------------------------------------------|
| `rfl`  | `0x0` | Flag register                             |
| `ridt` | `0x1` | IDT address                               |
| `racc` | `0x2` | Accumulator                               |
| `rcmp` | `0x3` | Extra register for conditional execution  |
| `rip`  | `0x4` | Instruction pointer                       |
| `rsp`  | `0x5` | Stack pointer                             |
| `r7`   | `0x7` | General purpose #6                        |
| ...    |       |                                           |
| `r15`  | `0xF` | General purpose #15                       |

Register `0x6` is reserved for future uses.

#### `rfl`

The flag register stores flags which are readable and writable by the program.  
It is read by the CPU (e.g. to control interrupts) and can be written to for certain flags (e.g. `_INTLOCK`).

| Name       | Offset\* | Description                          |
|------------|----------|--------------------------------------|
| `_INTON`   | `0`      | Interrupts globally enabled\*\*      |
| `_INT1ON`  | `1`      | Interrupt #1 enabled\*\*             |
| `_INT2ON`  | `2`      | Interrupt #2 enabled\*\*             |
| `_INT3ON`  | `3`      | Interrupt #3 enabled\*\*             |
| `_INT4ON`  | `4`      | Interrupt #4 enabled\*\*             |
| `_INTLOCK` | `5`      | Interrupt lock\*\*                   |

Flags at offsets `6` through `15` are reserved for future uses.

\*: Binary offset from the LSB, e.g. you can read `_INTLOCK` with `(rfl >> 5) & 0x1`.  
\*\*: See [IO and interrupts](io.md)

#### `ridt`

The IDT address refers to a table of interrupt handlers.  
See [IO and interrupts](io.md)

#### `racc`

The accumulator is mostly used as a destination register for arithmetic instructions.  
In the ntlabi it is also used as a return value in certain circumstances (see [Stack](stack.md)).

#### `rcmp`

The extra comparator operand is mostly used for the conditional execution of instructions.  
`racc` is still the main register for the conditional execution! This is only a secondary operand when the condition relies on a comparison between two registers.  
See [Instructions](instructions.md)

#### `rip`

The instruction pointer is the pointer to the currently running opcode.  
Writing to `rip` has no effect because it is just a clone of the internal program counter.

#### `r7`..`r15`

General purpose registers are never implicitly overriden by the CPU and has no special meaning, which makes them well-suited for intermediate results.  
However, they still should be saved and restored properly as required by the ntlabi, which will be used by most toolchains and programs for ntl, see [Stack](stack.md).