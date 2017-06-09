# **nanoThallium** (ntl) - RISC ISA

nanoThallium or ntl for short is a reduced instruction set computing architecture __(RISC)__. It is meant to be basic, easy to implement and easy to understand.

## Memory and registers

The 16 registers are __16-bit__ wide. Memory addressing is __16-bit__, and the memory atom type is __16-bit__, effectively enabling ~1Mbit memory. On the bootup, every register defaults to `0`.

ntl is a __load-store__ architecture, which means that memory access is done through special instructions. It is a __modified Harvard__ architecture, so the scratchpad memory and the program memory are separate, but can be accessed through separate instructions.

#### Register cheatsheet

| Name   | ID    | Description        |
|--------|-------|--------------------|
| `rfl`  | `0x0` | __FL__ag register  |
| `ridt` | `0x1` | __IDT__ address    |
| `racc` | `0x2` | __ACC__umulator    |

### Flag register

The `fl` register is the flag register, which stores some useful flags.

#### Flag register cheatsheet

| Name       | Offset | Description                        |
|------------|--------|------------------------------------|
| `_INTON`   | `0x0`  | __INT__errupts enabled             |
| `_INT1ON`  | `0x1`  | __INT__errupt #__1__ enabled       |
| `_INT2ON`  | `0x2`  | __INT__errupt #__2__ enabled       |
| `_INT3ON`  | `0x3`  | __INT__errupt #__3__ enabled       |
| `_INT4ON`  | `0x4`  | __INT__errupt #__4__ enabled       |
| `_INT5ON`  | `0x5`  | __INT__errupt #__5__ enabled       |
| `_INT6ON`  | `0x6`  | __INT__errupt #__6__ enabled       |
| `_INT7ON`  | `0x7`  | __INT__errupt #__7__ enabled       |
| `_TEST`    | `0x8`  | __TEST__ was positive              |
| `_IOFAIL`  | `0x9`  | __FAIL__ed __I__/__O__ transaction |
| `_INTLOCK` | `0xA`  | __INT__errupt __LOCK__             |

## Peripherials

#### Interrupts

The base ntl CPU can be woken up by __8__ interrupts. An interrupt stops the regular operation of the CPU and is triggered by an internal or external signal. They can be enabled or disabled globally through the `_INTON` flag and individually enabled through the `_INTxON` flags.

The `ridt` register is the offset to the Interrupt Descriptor Table. The IDT is a table with 8 entries of 16 bits. Each entry is an offset matching to an Interrupt Service Routine.

An ISR is a routine called when a matching interrupt is called. The return address matches to the instruction that should have been executed when being interrupted. However, there are things that the application programmer needs to be aware of:
- An ISR is a regular, non-returning function.
- Interrupts from ID `1` to `7` does not push any parameter data through the stack.
- Interrupt `0` is special and has extra things to take care of. _See: CPU exceptions_
- The interrupts does not affect registers. They should be saved and restored by the ISR to resume the task that was interrupted.
- When an interrupt rises, the `_INTLOCK` flag is set. It should be disabled by the ISR properly, because it temporarily disables interrupts.
- When interrupts are disabled (when `_INTON` and `_INT*ON` are not set), those interrupts are rejected and will never interrupt the CPU, even when reenabling interrupts.
- When interrupts are enabled, but `_INTLOCK` is set, the interrupts will stack in a dedicated interrupt stack. This stack has a limited size, so pushing an interrupt to the interrupt stack will effectively reject the interrupt.

##### CPU exceptions (Interrupt `0`)

A CPU exception is a special interrupt that cannot be disabled. Unlike other interrupts, it requires a special ISR to handle the CPU exception ID, which is pushed on the stack as a parameter. When a CPU exception rises while `_INTLOCK` or `_INTON` is set, the CPU will halt. The following CPU exception IDs exist:

| Name          | ID       |
|---------------|----------|
| `_ARITHMETIC` | `0x0000` |

#### Port I/O

The base ntl CPU has access to __65'536__ ports. Ports can be used to send and receive 8 bits (half a register) a clock, and can be used to communicate with external, non-standard peripherials.

Port I/O is performed through the `read` and `write` instructions. Those operations are non-blocking so a transaction failing at a given clock time will result into the `_IOFAIL` flag being set.

## Instruction set

#### Instruction encoding

#### Opcode cheatsheet

| Name       | Arguments     | ID     | Description                                   | Timing* |
|------------|---------------|--------|-----------------------------------------------|---------|
| `nop`      |               | `0x00` | __NO__ o__P__eration                          | `1`     |
| `load`     | `rdst, raddr` | `0x01` | __LOAD__ from memory                          | `2`     |
| `store`    | `rsrc, raddr` | `0x02` | __STORE__ to memory                           | `2`     |
| `pload`    | `rdst, raddr` | `0x03` | __LOAD__ from __P__rogram memory              | `2`     |
| `pstore`   | `rsrc, raddr` | `0x04` | __STORE__ to __P__rogram memory               | `2`     |
| `mov`      | `rsrc, rdst`  | `0x05` | __MOV__e register                             | `1`     |
| `add`\*\*  | `ra, rb`      | `0x06` | __ADD__                                       | `1`     |
| `sub`\*\*  | `ra, rb`      | `0x07` | __SUB__tract                                  | `1`     |
| `mul`\*\*  | `ra, rb`      | `0x08` | __MUL__tiply                                  | TBD     |
| `div`\*\*  | `ra, rb`      | `0x09` | __DIV__ide                                    | TBD     |
| `and`\*\*  | `ra, rb`      | `0x0A` | Bitwise __AND__                               | `1`     |
| `or`\*\*   | `ra, rb`      | `0x0B` | Bitwise __OR__                                | `1`     |
| `xor`\*\*  | `ra, rb`      | `0x0C` | Bitwise __XOR__                               | `1`     |
| `not`      | `ra, rdst`    | `0x0D` | Bitwise __NOT__                               | `1`     |
| `shl`\*\*  | `ra, roff`    | `0x0E` | Bitwise __SH__ift __L__eft                    | `1`     |
| `shr`\*\*  | `ra, roff`    | `0x0F` | Bitwise __SH__ift __R__ight                   | `1`     |
| `ashr`\*\* | `ra, roff`    | `0x10` | __A__rithmetic __SH__ift __R__ight            | `1`     |
| `gbit`     | `ra, roff`    | `0x11` | __G__et __BIT__ at offset to LSD              | `1`     |
| `fbit`     | `ra, roff`    | `0x12` | __F__lip __BIT__ at offset                    | `1`     |
| `pop`      | `rdst`        | `0x13` | __POP__ register from stack                   | `2`     |
| `push`     | `rsrc`        | `0x14` | __PUSH__ register to stack                    | `2`     |
| `jmpi`     | `iaddr`       | `0x15` | __J__u__MP__ to __I__mmediate                 | `2`     |
| `jmp`      | `raddr`       | `0x16` | __J__u__MP__                                  | `2`     |
| `cjmpi`    | `iaddr`       | `0x17` | __C__onditional __J__u__MP__ to __I__mmediate | `2`     |
| `cjmp`     | `raddr`       | `0x18` | __C__onditional __J__u__MP__                  | `2`     |
| `ret`      |               | `0x19` | __RET__urn                                    | `2`     |
| `calli`    | `iaddr`       | `0x1A` | __CALL I__mmediate function                   | TBD     |
| `call`     | `raddr`       | `0x1B` | __CALL__ function                             | TBD     |
| `tz`       | `ra`          | `0x1C` | __T__est: equal to __Z__ero                   | `1`     |
| `tht`      | `ra, rb`      | `0x1D` | __T__est: __H__igher __T__han                 | `1`     |
| `thq`      | `ra, rb`      | `0x1E` | __T__est: __H__igher or e__Q__ual to          | `1`     |
| `teq`      | `ra, rb`      | `0x1F` | __T__est: __EQ__ual to                        | `1`     |
| `ldi`      | `ia`          | `0x20` | __L__oa__D__ __I__mmediate                    | `1`     |
| `hlt`      |               | `0x21` | __H__a__LT__ CPU                              | `1`     |
| `read`     | `rdst, rport` | `0x22` | I/O __READ__                                  | `2`     |
| `write`    | `rsrc, rport` | `0x23` | I/O __WRITE__                                 | `1`     |
| `wait`     | `rport`       | `0x24` | I/O port __WAIT__                             | `1`     |
| `int`      | `iid`         | `0x25` | Throw fake __INT__errupt                      | TBD     |

\* Specified timings are the reference timing for the base VHDL implementation of ntl.  
\*\* Arithmetic instructions such as `add` with two operands stores their result to the `racc` register.

#### Opcode detailed information

- ##### `nop` (`0x00`)

No operation instruction. Does not alter the CPU state.

- ##### `load rdst, raddr` (`0x01`)

Load from scratchpad memory.  
Loads the 16-bit word at memory address `raddr` and stores it to `rdst`.

- ##### `store rsrc, raddr` (`0x02`)

Store to scratchpad memory.  
Stores the `rsrc` register to the word at memory address `raddr`.

- ##### `pload rdst, raddr` (`0x03`)

Load from program memory.  
Loads the 16-bit instruction at memory address `raddr` and stores it to `rdst`.

- ##### `pstore rsrc, raddr` (`0x04`)

Store to program memory.  
Stores the `rsrc register` to the instruction at memory address `raddr`.

- ##### `mov rsrc, rdst` (`0x05`)

Copy the register `rsrc` content to `rdst`.

- ##### `add ra, rb` (`0x06`)

Integer addition.  
Perform `ra + rb` and store to `racc`.

- ##### `sub ra, rb` (`0x07`)

Integer subtraction.  
Perform `ra - rb` and store to `racc`.

- ##### `mul ra, rb` (`0x08`)

Integer multiplication.  
Perform `ra * rb` and store to `racc`.

- ##### `div ra, rb` (`0x09`)

Integer division.  
Perform `ra / rb` and store to `racc`.  
If `rb` is zero, a `_ARITHMETIC` CPU exception will rise.

- ##### `and ra, rb` (`0x0A`)

Bitwise AND.  
Perform `ra & rb` and store to `racc`.

- ##### `or ra, rb` (`0x0B`)

Bitwise OR.  
Perform `ra | rb` and store to `racc`.

- ##### `xor ra, rb` (`0x0C`)

Bitwise exclusive OR (XOR).  
Perform `ra ^ rb` and store to `racc`.  
Using `xor ra, ra` will effectively clear the register.

- ##### `not ra, rdst` (`0x0D`)

Bitwise NOT.  
Perform `~ra` and store to `rdst`.

- ##### `shl ra, roff` (`0x0E`)

Bitwise bitshift left.  
Perform `ra << roff` and store to `racc`.  
The bits appearing on the right are `0`.

- ##### `shr ra, roff` (`0x0F`)

Bitwise bitshift left.  
Perform `ra << roff` and store to `racc`.  
The bits appearing on the left are `0`.

- ##### `ashr ra, roff` (`0x10`)

Arithmetic bitshift right.  
It behaves similarly to a bitwise bitshift right, but the MSB is copied over the `racc` destination.

- ##### `gbit ra, roff` (`0x11`)

Get bit at offset.  
The bit of `ra` at offset `roff` is copied to the LSB of `racc`.

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

- ##### `ldi ia` (`0x20`)

Load immediate value.  
Stores the `ia` value to `racc`.

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

## Stack

#### Functions

## Assembly language

#### Notation

Identifiers are case sensitive. An user identifier can begin with any alphanumerical character. Only built-in identifiers, such as flag offsets, may begin by `_`.  
Register names are prefixed by `r`.

#### Instructions

An instruction is defined by writing its name, followed by a space and an argument list, if required. Arguments in the argument list are separated by a comma `,`. For example:

`mov racc, r0`

This is read as a `mov` instruction, with the `racc` register as the first operand and `r0` as the second operand. When the amount of arguments in the argument list does not match to what the instruction expects, an error occurs.

Both newlines and semicolons `;` may separate instructions. In this case:

    mov racc, r0
    mov racc, r1

is interpreted the same way as

    mov racc, r0; mov racc, r1

Empty instructions are ignored, so `;;;;` is not a syntax error and will emit _nothing_ (not the `nop` instruction either).

#### Comments

Comments are pieces of text that are ignored by the assembler and not parsed in any way. Comments can begin anywhere in a line from the `#` character and finishes at the end of this line.

    # this is a comment
    # this is another comment
    mov racc, r0 # explain something complicated
    mov racc, r1
    # mov racc, r2 - commented instructions are ignored

#### Labels

Labels denotes a memory offset to the program space to the following instruction or assembler directive, if applicable. Label names are prefixed by `%` and are user defined.  
These expands into a 16-bit memory address, that can be directly used as an immediate argument for an instruction such as `calli` or `jmpi`.

    %infiniteLoop
      jmpi %infiniteLoop

#### Assembler directives

Assembler directives allows using assembly or assembler related features not available otherwise. These are prefixed by `.`. For example, the `.at` directive forces the following instruction to be located at that specific area.

| Name  | Description                           |
|-------|---------------------------------------|
| `.at` | Force the assembler location counter. |

- ##### `.at`

Forces the assembler location counter.  
In other words, the next instruction (or assembler directive) will be located at the given offset in the resulting flat binary.

    .at 0x1000
    mov racc, r0
    mov racc, r1

This piece of code will have the `mov racc, r0` instruction located at `0x1000` in the flat binary. Do remember that the memory word is 16-bit in ntl, so the actual byte offset would be `0x2000`.  
The instruction that follows naturally is located at `0x1002`.

#### Instruction aliases

Some instruction aliases are exposed to the assembler for convenience. They are always prefixed by `$` to differenciate them from regular instructions, and may emit one or more instructions.

| Alias    | Alias arguments | Aliased instructions          |
|----------|-----------------|-------------------------------|
| `$tnz`   | `ra`            | `tz ra; fbit rfl, _TEST`      |
| `$tlt`   | `ra, rb`        | `tht ra, rb; fbit rfl, _TEST` |
| `$tlq`   | `ra, rb`        | `tlq ra, rb; fbit rfl, _TEST` |
| `$tnq`   | `ra, rb`        | `teq ra, rb; fbit rfl, _TEST` |

#### Warnings and errors
