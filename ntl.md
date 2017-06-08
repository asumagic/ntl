# **nanoThallium** (ntl) - MISC ISA

nanoThallium or ntl for short is a reduced instruction set computing architecture __(RISC)__. It is meant to be basic, easy to implement and easy to understand.

## Memory and registers

The 16 registers are __16-bit__ wide. Memory addressing is __16-bit__, and the memory atom type is __16-bit__, effectively enabling ~1Mbit memory. On the bootup, every register defaults to `0`.

ntl is a __load-store__ architecture, which means that memory access is done through special instructions. It is a __modified Havard__ architecture, so the scratchpad memory and the program memory are separate, but can be accessed through separate instructions.

#### Register cheatsheet

| Name | ID    | Description        |
|------|-------|--------------------|
| `fl` | `0x0` | __FL__ag register  |

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

## Peripherials

#### Interrupts

The base ntl CPU has access to __8__ interrupts. An interrupt stops the regular operation of the CPU and is triggered by an internal or external signal. They can be enabled or disabled globally through the `_INTON` flag and individually enabled through the `_INTxON` flags.

The `int` register is the offset to the Interrupt Descriptor Table. The IDT is a table with 8 entries of 16 bits. Each entry is an offset matching to an Interrupt Service Routine.

An ISR is a routine called when a matching interrupt is called. The return address matches to the instruction that should have been executed when being interrupted. However, there are things that the application programmer needs to be aware of:
- An ISR is a regular, non-returning function.
- Interrupts from ID `1` to `7` does not push any parameter data through the stack.
- Interrupt `0` passes the CPU exception ID to the stack.
- Interrupt `0` cannot be disabled. If a CPU exception occurs when `_INTON` is disabled, the CPU will halt.
- The interrupts does not affect registers. They should be saved and restored by the ISR to resume the task interrupted.

## Instruction set

#### Instruction encoding

#### Opcode cheatsheet

| Name       | Arguments     | ID     | Description                      | Timing* |
|------------|---------------|--------|----------------------------------|---------|
| `nop`      |               | `0x00` | __NO__ o__P__eration             | `1`     |
| `load`     | `rdst, raddr` | `0x01` | __LOAD__ from memory             | `2`     |
| `store`    | `rsrc, raddr` | `0x02` | __STORE__ to memory              | `2`     |
| `pload`    | `rdst, raddr` | `0x03` | __LOAD__ from __P__rogram memory | `2`     |
| `pstore`   | `rsrc, raddr` | `0x04` | __STORE__ to __P__rogram memory  | `2`     |
| `mov`      | `rsrc, rdst`  | `0x05` | __MOV__e register                | `1`     |

\* Specified timings are the reference timing for the base VHDL implementation of ntl.

## Stack

#### Functions

## Assembly language

#### Keywords

Every identifier that begins with an underscore `_` is reserved.
