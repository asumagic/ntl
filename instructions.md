---
title: Instructions
---

Instructions
=====

#### Instruction encoding

Instructions have a fixed size of **32** bits (2 CPU words). However, the internal structure of the instruction may vary.  
The opcode always appears in the first byte, but the other operands may not be used by instructions. In this case, the implementation will ignore these bits.  

| Name                  | Range          |
|-----------------------|----------------|
| Opcode                | `0x00`->`0x07` |
| Register operand `#1` | `0x08`->`0x0B` |
| Register operand `#2` | `0x0C`->`0x0F` |
| Immediate             | `0x10`->`0x1F` |


#### Opcode cheatsheet

| Mnemonic   | Arguments             | ID     | Description                                   |
|------------|-----------------------|--------|-----------------------------------------------|
| `nop`      |                       | `0x00` | No operation                                  |
| `load`     | `rdst, raddr`         | `0x01` | Load from memory                              |
| `store`    | `rsrc, raddr`         | `0x02` | Store to memory                               |
| `mov`      | `rsrc, rdst`          | `0x03` | Copy register value                           |
| `add`      | `ra, rb`              | `0x04` | Integer addition                              |
| `sub`      | `ra, rb`              | `0x05` | Integer subtraction                           |
| `mul`      | `ra, rb`              | `0x06` | Integer multiplication                        |
| `div`      | `ra, rb`              | `0x07` | Integer division                              |
| `and`      | `ra, rb`              | `0x08` | Bitwise AND                                   |
| `or`       | `ra, rb`              | `0x09` | Bitwise OR                                    |
| `xor`      | `ra, rb`              | `0x0A` | Bitwise XOR                                   |
| `not`      | `ra`                  | `0x0B` | Bitwise NOT                                   |
| `shl`      | `ra, roff`            | `0x0C` | Bitwise left shift                            |
| `shr`      | `ra, roff`            | `0x0D` | Bitwise right shift                           |
| `gbit`     | `ra, roff`            | `0x0E` | Copy bit at offset to LSB                     |
| `fbit`     | `ra, roff`            | `0x0F` | Flip bit at offset                            |
| `pop`      | `rdst`                | `0x10` | Pop stack value to register                   |
| `push`     | `rsrc`                | `0x11` | Push register value to stack                  |
| `jmpi`     | `iaddr`               | `0x12` | Jump to immediate address                     |
| `jmp`      | `raddr`               | `0x13` | Jump to register value address                |
| `cjmpi`    | `iaddr`               | `0x14` | Conditional jump to immediate address         |
| `cjmp`     | `raddr`               | `0x15` | Conditional jump to register value address    |
| `ret`      |                       | `0x16` | Return from function                          |
| `calli`    | `iaddr`               | `0x17` | Call to immediate address function            |
| `call`     | `raddr`               | `0x18` | Call to register value address function       |
| `tz`       | `ra`                  | `0x19` | Test equality to zero                         |
| `tht`      | `ra, rb`              | `0x1A` | Test for higher than                          |
| `thq`      | `ra, rb`              | `0x1B` | Test for higher than or equal to              |
| `teq`      | `ra, rb`              | `0x1C` | Test for equal to                             |
| `ldi`      | `rdst, ia`            | `0x1D` | Load immediate value to register              |
| `hlt`      |                       | `0x1E` | Halt CPU until interrupt                      |
| `int`      | `iid`                 | `0x1F` | Raise interrupt                               |

#### Opcode detailed information

- ##### `nop`

No operation instruction. Does not alter the CPU state.

- ##### `load rdst, raddr`

Loads the 16-bit word at memory address `raddr` in memory and stores it to `rdst`.

- ##### `store rsrc, raddr`

Stores the `rsrc` register to the word at memory address `raddr` in memory.

- ##### `mov rsrc, rdst`

Copy the register value `rsrc` to `rdst`.

- ##### `add ra, rb`

Integer addition.  
Perform `ra + rb` and store to `racc`.

- ##### `sub ra, rb`

Integer subtraction.  
Perform `ra - rb` and store to `racc`.

- ##### `mul ra, rb`

Integer multiplication.  
Perform `ra * rb` and store to `racc`.

- ##### `div ra, rb`

Integer division.  
Perform `ra / rb` and store to `racc`.  
If `rb` is zero, a `_ARITHMETIC` CPU exception will rise.

- ##### `and ra, rb`

Bitwise AND.  
Perform `ra & rb` and store to `racc`.

- ##### `or ra, rb`

Bitwise OR.  
Perform `ra | rb` and store to `racc`.

- ##### `xor ra, rb`

Bitwise exclusive OR (XOR).  
Perform `ra ^ rb` and store to `racc`.  
Using `xor ra, ra` can be used for clearing the register.

- ##### `not ra, rdst`

Bitwise NOT.  
Perform `~ra` and store to `racc`.

- ##### `shl ra, roff`

Bitwise bitshift left.  
Perform `ra << roff` and store to `racc`.  
`0` is used for padding.

- ##### `shr ra, roff`

Bitwise bitshift right.  
Perform `ra >> roff` and store to `rdst`.  
`0` is used for padding.

- ##### `gbit ra, roff`

Get bit at offset.  
The bit of `ra` at offset `roff` is copied to the LSB of `racc`.

- ##### `fbit ra, roff`

Flip bit at offset.  
The bit of `ra` at offset `roff` is flipped (`0`->`1` and `1`->`0`).  
The result is stored to `ra`.

- ##### `pop rdst`

Pop stack value to register.  
`load` word at address `rsp` from scratchpad memory to `rdst` and decrement `rsp`.

- ##### `push rsrc`

Push register to stack.  
Increment `rsp` and `store` the `rsrc` register to the new `rsp` memory address.

- ##### `jmpi iaddr`

Jump to immediate address.  
Sets the read-only `ip` register to `iaddr`, which is the address to the next instruction executed.

- ##### `jmp raddr`

Jump to register-defined address.  
Sets the read-only `ip` register to `raddr`, which is the address to the next instruction executed.

- ##### `cjmpi iaddr`

Conditional jump to immediate address.  
If the `_TEST` flag is set, the jump will be performed, and the `_TEST` flag will be cleared.

_See: `jmpi`_

- ##### `cjmp raddr`

Conditional jump to register-defined address.  
If the `_TEST` flag is set, the jump will be performed, and the `_TEST` flag will be cleared.

_See: `jmp`_

- ##### `ret`

Return from function.  
Pop stack value to `ip` and increment it, so it becomes the next instruction to execute.

_See: Functions_

- ##### `calli iaddr`

Call function at immediate address.  
Push `ip` to the stack and `jmpi iaddr`.

_See: Functions_

- ##### `call raddr`

Call function at register-defined address.  
Push `ip` to the stack and `jmpi iaddr`.

_See: Functions_

- ##### `tz ra`

Test for equality to zero.  
Sets the `_TEST` flag if `ra == 0`.  
When the condition is not met, the `_TEST` flag *will be left untouched*.

- ##### `tht ra, rb`

Test for higher than.  
Sets the `_TEST` flag if `ra > rb`.  
When the condition is not met, the `_TEST` flag *will be left untouched*.

- ##### `thq ra, rb`

Test for higher or equal to.  
Sets the `_TEST` flag if `ra >= rb`.  
When the condition is not met, the `_TEST` flag *will be left untouched*.

- ##### `teq ra, rb`

Test for equality.  
Sets the `_TEST` flag if `ra == rb`.  
When the condition is not met, the `_TEST` flag *will be left untouched*.

- ##### `ldi rdst, ia`

Load immediate value.  
Stores the `ia` value to `rdst`.

- ##### `hlt`

Halt the CPU and wait for an interrupt.  
If interrupts are globally disabled, the CPU will halt and catch fire.

- ##### `int iid`

Raise a user interrupt with the 3 lower bits of `iid` as ID.  
Other bits are ignored by the CPU.

_See: Interrupts_
