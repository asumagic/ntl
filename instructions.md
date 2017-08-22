## Instructions

#### Instruction encoding

Instructions have a fixed size of **32** bits (2 CPU words). Instruction memory addressing has a word size of 16 bits, similarly to the scratchpad memory. However, the internal structure of the instruction may vary. If the opcode always appears in the first byte, the other operands may not be used by instructions. In this case, the CPU will ignore these bits.  
Some areas like the register operand `#3` range overlaps the immediate range, in which case the instruction will not use both.

| Name                  | Range          |
|-----------------------|----------------|
| Opcode                | `0x00`->`0x07` |
| Register operand `#1` | `0x08`->`0x0B` |
| Register operand `#2` | `0x0C`->`0x0F` |
| Register operand `#3` | `0x10`->`0x13` |
| Immediate             | `0x10`->`0x1F` |


#### Opcode cheatsheet

| Mnemonic   | Arguments             | ID     | Description                                   | Timing* |
|------------|-----------------------|--------|-----------------------------------------------|---------|
| `nop`      |                       | `0x00` | __NO__ o__P__eration                          | `1`     |
| `load`     | `rdst, raddr`         | `0x01` | __LOAD__ from memory                          | `2`     |
| `store`    | `rsrc, raddr`         | `0x02` | __STORE__ to memory                           | `2`     |
| `pload`    | `rdst, raddr`         | `0x03` | __LOAD__ from __P__rogram memory              | `2`     |
| `pstore`   | `rsrc, raddr`         | `0x04` | __STORE__ to __P__rogram memory               | `2`     |
| `mov`      | `rsrc, rdst`          | `0x05` | __MOV__e register                             | `1`     |
| `add`      | `ra, rb, rdst`        | `0x06` | __ADD__                                       | `1`     |
| `sub`      | `ra, rb, rdst`        | `0x07` | __SUB__tract                                  | `1`     |
| `mul`      | `ra, rb, rdst`        | `0x08` | __MUL__tiply                                  | TBD     |
| `div`      | `ra, rb, rdst`        | `0x09` | __DIV__ide                                    | TBD     |
| `and`      | `ra, rb, rdst`        | `0x0A` | Bitwise __AND__                               | `1`     |
| `or`       | `ra, rb, rdst`        | `0x0B` | Bitwise __OR__                                | `1`     |
| `xor`      | `ra, rb, rdst`        | `0x0C` | Bitwise __XOR__                               | `1`     |
| `not`      | `ra, rdst`            | `0x0D` | Bitwise __NOT__                               | `1`     |
| `shl`      | `ra, roff, rdst`      | `0x0E` | Bitwise __SH__ift __L__eft                    | `1`     |
| `shr`      | `ra, roff, rdst`      | `0x0F` | Bitwise __SH__ift __R__ight                   | `1`     |
| `ashr`     | `ra, roff, rdst`      | `0x10` | __A__rithmetic __SH__ift __R__ight            | `1`     |
| `gbit`     | `ra, roff, rdst`      | `0x11` | __G__et __BIT__ at offset to LSD              | `1`     |
| `fbit`     | `ra, roff`            | `0x12` | __F__lip __BIT__ at offset                    | `1`     |
| `pop`      | `rdst`                | `0x13` | __POP__ register from stack                   | `2`     |
| `push`     | `rsrc`                | `0x14` | __PUSH__ register to stack                    | `2`     |
| `jmpi`     | `iaddr`               | `0x15` | __J__u__MP__ to __I__mmediate                 | `2`     |
| `jmp`      | `raddr`               | `0x16` | __J__u__MP__                                  | `2`     |
| `cjmpi`    | `iaddr`               | `0x17` | __C__onditional __J__u__MP__ to __I__mmediate | `2`     |
| `cjmp`     | `raddr`               | `0x18` | __C__onditional __J__u__MP__                  | `2`     |
| `ret`      |                       | `0x19` | __RET__urn                                    | `2`     |
| `calli`    | `iaddr`               | `0x1A` | __CALL I__mmediate function                   | TBD     |
| `call`     | `raddr`               | `0x1B` | __CALL__ function                             | TBD     |
| `tz`       | `ra`                  | `0x1C` | __T__est: equal to __Z__ero                   | `1`     |
| `tht`      | `ra, rb`              | `0x1D` | __T__est: __H__igher __T__han                 | `1`     |
| `thq`      | `ra, rb`              | `0x1E` | __T__est: __H__igher or e__Q__ual to          | `1`     |
| `teq`      | `ra, rb`              | `0x1F` | __T__est: __EQ__ual to                        | `1`     |
| `ldi`      | `rdst, ia`            | `0x20` | __L__oa__D__ __I__mmediate                    | `1`     |
| `hlt`      |                       | `0x21` | __H__a__LT__ CPU                              | `1`     |
| `read`     | `rdst, rport`         | `0x22` | I/O __READ__                                  | `2`     |
| `write`    | `rsrc, rport`         | `0x23` | I/O __WRITE__                                 | `1`     |
| `wait`     | `rport`               | `0x24` | I/O port __WAIT__                             | `1`     |
| `int`      | `iid`                 | `0x25` | Throw fake __INT__errupt                      | TBD     |

\* Specified timings are the reference timing for the base VHDL implementation of ntl.  

#### Opcode detailed information

- ##### `nop` (`0x00`)

No operation instruction. Does not alter the CPU state.

- ##### `load rdst, raddr` (`0x01`)

Load from scratchpad memory.  
Loads the 16-bit word at memory address `raddr` in scratchpad memory and stores it to `rdst`.

- ##### `store rsrc, raddr` (`0x02`)

Store to scratchpad memory.  
Stores the `rsrc` register to the word at memory address `raddr` in scratchpad memory.

- ##### `pload rsrc, raddr` (`0x03`)

Load from program memory.  
Loads the 16-bit word at memory address `raddr` in program memory and it to `rdst`.

- ##### `pstore rdst, raddr` (`0x04`)

Store to program memory.  
Stores to the `rsrc` register to the word at memory address `raddr` in program memory.

- ##### `mov rsrc, rdst` (`0x05`)

Copy the register `rsrc` content to `rdst`.

- ##### `add ra, rb, rdst` (`0x06`)

Integer addition.  
Perform `ra + rb` and store to `rdst`.

- ##### `sub ra, rb, rdst` (`0x07`)

Integer subtraction.  
Perform `ra - rb` and store to `rdst`.

- ##### `mul ra, rb, rdst` (`0x08`)

Integer multiplication.  
Perform `ra * rb` and store to `rdst`.

- ##### `div ra, rb, rdst` (`0x09`)

Integer division.  
Perform `ra / rb` and store to `rdst`.  
If `rb` is zero, a `_ARITHMETIC` CPU exception will rise.

- ##### `and ra, rb, rdst` (`0x0A`)

Bitwise AND.  
Perform `ra & rb, rdst` and store to `rdst`.

- ##### `or ra, rb` (`0x0B`)

Bitwise OR.  
Perform `ra | rb` and store to `rdst`.

- ##### `xor ra, rb, rdst` (`0x0C`)

Bitwise exclusive OR (XOR).  
Perform `ra ^ rb` and store to `rdst`.  
Using `xor ra, ra` will effectively clear the register.

- ##### `not ra, rdst` (`0x0D`)

Bitwise NOT.  
Perform `~ra` and store to `rdst`.

- ##### `shl ra, roff, rdst` (`0x0E`)

Bitwise bitshift left.  
Perform `ra << roff` and store to `rdst`.  
The bits appearing on the right are `0`.

- ##### `shr ra, roff, rdst` (`0x0F`)

Bitwise bitshift left.  
Perform `ra << roff` and store to `rdst`.  
The bits appearing on the left are `0`.

- ##### `ashr ra, roff, rdst` (`0x10`)

Arithmetic bitshift right.  
It behaves similarly to a bitwise bitshift right, but the MSB is copied over the `rdst` destination.

- ##### `gbit ra, roff, rdst` (`0x11`)

Get bit at offset.  
The bit of `ra` at offset `roff` is copied to the LSB of `rdst`.

- ##### `fbit ra, roff` (`0x12`)

Flip bit at offset.  
The bit of `ra` at offset `roff` is flipped (`0`->`1` and `1`->`0`)

- ##### `pop rdst` (`0x13`)

Pop stack value to register.  
`load` word at address `rsp` from scratchpad memory to `rdst` and decrement `rsp`.

- ##### `push rsrc` (`0x14`)

Push register to stack.  
Increment `rsp` and `store` the `rsrc` register to the new `rsp` memory address.

- ##### `jmpi iaddr` (`0x15`)

Jump to immediate address.  
Sets the read-only `ip` register to `iaddr`, which is the address to the next instruction executed.

- ##### `jmp raddr` (`0x16`)

Jump to register-defined address.  
Sets the read-only `ip` register to `raddr`, which is the address to the next instruction executed.

- ##### `cjmpi iaddr` (`0x17`)

Conditional jump to immediate address.  
If the `_TEST` flag is set, the jump will be performed, and the `_TEST` flag will be cleared.

_See: `jmpi`_

- ##### `cjmp raddr` (`0x18`)

Conditional jump to register-defined address.  
If the `_TEST` flag is set, the jump will be performed, and the `_TEST` flag will be cleared.

_See: `jmp`_

- ##### `ret` (`0x19`)

Return from function.  
Pop stack value to `ip` and increment it, so it becomes the next instruction to execute.

_See: Functions_

- ##### `calli iaddr` (`0x1A`)

Call function at immediate address.  
Push `ip` to the stack and `jmpi iaddr`.

_See: Functions_

- ##### `call raddr` (`0x1B`)

Call function at register-defined address.  
Push `ip` to the stack and `jmpi iaddr`.

_See: Functions_

- ##### `tz ra` (`0x1C`)

Test for equality to zero.  
Sets the `_TEST` flag if `ra == 0`.

- ##### `tht ra, rb` (`0x1D`)

Test for higher than.  
Sets the `_TEST` flag if `ra > rb`.

- ##### `thq ra, rb` (`0x1E`)

Test for higher or equal to.  
Sets the `_TEST` flag if `ra >= rb`.

- ##### `teq ra, rb` (`0x1F`)

Test for equality.  
Sets the `_TEST` flag if `ra == rb`.

- ##### `ldi rdst, ia` (`0x20`)

Load immediate value.  
Stores the `ia` value to `rdst`.

- ##### `hlt` (`0x21`)

Halt the CPU and wait for an interrupt.
If interrupts are globally disabled, the CPU will halt and catch fire.

- ##### `read rdst, rport` (`0x22`)

Non-blocking I/O port read.  
Read a byte from `rport` and write it to the lower byte of `rdst`.  
When the port does not exist or when no data is available at the port, the lower byte of `rdst` is not modified and the `_IOFAIL` flag is set.

- ##### `write rsrc, rport` (`0x23`)

Non-blocking I/O port write.  
Write the lower byte of `rdst` to the `rport` port.
When the port does not exist or when the port cannot receive data at the moment, the `_IOFAIL` flag is set.

- ##### `wait rport` (`0x24`)

I/O port wait.  
Halt the CPU until there is data to read from `rport`.

- ##### `int iid` (`0x25`)

Throw a fake user interrupt with the 3 lower bits of `iid` as ID.

_See: Interrupts_
