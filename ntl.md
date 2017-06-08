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

| Name      | Offset | Description                  |
|-----------|--------|------------------------------|
| `_INTON`  | `0x0`  | __INT__errupts enabled       |
| `_INT1ON` | `0x1`  | __INT__errupt #__1__ enabled |
| `_INT2ON` | `0x2`  | __INT__errupt #__2__ enabled |
| `_INT3ON` | `0x3`  | __INT__errupt #__3__ enabled |
| `_INT4ON` | `0x4`  | __INT__errupt #__4__ enabled |
| `_INT5ON` | `0x5`  | __INT__errupt #__5__ enabled |
| `_INT6ON` | `0x6`  | __INT__errupt #__6__ enabled |
| `_INT7ON` | `0x7`  | __INT__errupt #__7__ enabled |
| `_TEST`   | `0x8`  | __TEST__ was positive        |

## Peripherials

#### Interrupts

The base ntl CPU has access to __8__ interrupts. An interrupt stops the regular operation of the CPU and is triggered by an internal or external signal. They can be enabled or disabled globally through the `_INTON` flag and individually enabled through the `_INTxON` flags.

The `ridt` register is the offset to the Interrupt Descriptor Table. The IDT is a table with 8 entries of 16 bits. Each entry is an offset matching to an Interrupt Service Routine.

An ISR is a routine called when a matching interrupt is called. The return address matches to the instruction that should have been executed when being interrupted. However, there are things that the application programmer needs to be aware of:
- An ISR is a regular, non-returning function.
- Interrupts from ID `1` to `7` does not push any parameter data through the stack.
- Interrupt `0` passes the CPU exception ID to the stack.
- Interrupt `0` cannot be disabled. If a CPU exception occurs when `_INTON` is disabled, the CPU will halt.
- The interrupts does not affect registers. They should be saved and restored by the ISR to resume the task interrupted.

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
| `shl`\*\*  | `ra, roff`    | `0x0C` | Bitwise __SH__ift __L__eft                    | `1`     |
| `shr`\*\*  | `ra, roff`    | `0x0D` | Bitwise __SH__ift __R__ight                   | `1`     |
| `ashr`\*\* | `ra, roff`    | `0x0E` | __A__rithmetic __SH__ift __R__ight            | `1`     |
| `gbit`\*\* | `ra, roff`    | `0x0F` | __G__et __BIT__ at offset to LSD              | `1`     |
| `fbit`\*\* | `ra, roff`    | `0x0A` | __F__lip __BIT__ at offset                    | `1`     |
| `pop`      | `rdst`        | `0x0B` | __POP__ register from stack                   | `2`     |
| `push`     | `rsrc`        | `0x0C` | __PUSH__ register to stack                    | `2`     |
| `jmpi`     | `iaddr`       | `0x0D` | __J__u__MP__ to __I__mmediate                 | `1`     |
| `jmp`      | `raddr`       | `0x0E` | __J__u__MP__                                  | `1`     |
| `cjmpi`    | `iaddr`       | `0x0F` | __C__onditional __J__u__MP__ to __I__mmediate | `1`     |
| `cjmp`     | `raddr`       | `0x10` | __C__onditional __J__u__MP__                  | `1`     |
| `ret`      |               | `0x11` | __RET__urn from function                      | `1`     |
| `tz`       | `ra`          | `0x12` | __T__est: equal to __Z__ero                   | `1`     |
| `tht`      | `ra, rb`      | `0x13` | __T__est: __H__igher __T__han                 | `1`     |
| `thq`      | `ra, rb`      | `0x14` | __T__est: __H__igher or e__Q__ual to          | `1`     |
| `teq`      | `ra, rb`      | `0x15` | __T__est: __EQ__ual to                        | `1`     |
| `ldi`      | `ia`          | `0x16` | __L__oa__D__ __I__mmediate                    | `1`     |
| `hlt`      |               | `0x17` | __H__a__LT__ CPU                              | `1`     |

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

- ##### `hlt` (`0x17`)

Halt the CPU and wait for an interrupt.  
If interrupts are globally disabled, the CPU will halt and catch fire.

## Stack

#### Functions

## Assembly language

#### Notation

Identifiers are case sensitive. An user identifier can begin with any alphanumerical character. Built-in identifiers, such as flag offsets, may begin by `_`.  
Register names are prefixed by `r`.

#### Keywords
