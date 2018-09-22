---
title: Instructions
---

[Jump to main page](README.md)

Instructions
=====

## Instruction encoding

Opcodes are always encoded through **24** bits, but their structure varies.  

| Bits       | Name          | Description                                                                             |
|------------|---------------|-----------------------------------------------------------------------------------------|
|  `0`..`2`  | Conditional   | Conditional execution instruction.                                                      |
|  `3`..`7`  | Instruction   | Type of executed instruction.                                                           |
|  `8`..`11` | Register A    | (Optional) Read-write register operand.                                                 |
| `12`..`15` | Register B    | (Optional) Write-only register operand.                                                 |
|  `8`..`23` | Immediate     | (Optional) Arbitrary value to be used by the instruction.                               |

## Conditional execution

The conditional execution instruction determines whether an instruction should be executed or not, based on a given condition.  
This generally involves the `racc` and `rcmp` registers.  
When the condition is not met, the CPU skips to the next opcode without any side effects.

| Mnemonic   | ID            | Allows execution when                         | Cycles |
|------------|---------------|-----------------------------------------------|--------|
| (none)     | `0x0`/`0b000` | Always                                        | `0`    |
| `?z`       | `0x1`/`0b001` | `racc` = 0                                    | TBD    |
| `?nz`      | `0x2`/`0b010` | `racc` != 0                                   | TBD    |
| `?eq`      | `0x3`/`0b011` | `racc` = `rcmp`                               | TBD    |
| `?neq`     | `0x4`/`0b100` | `racc` != `rcmp`                              | TBD    |
| `?bt`      | `0x5`/`0b101` | `racc` > `rcmp`                               | TBD    |
| `?bet`     | `0x6`/`0b110` | `racc` >= `rcmp`                              | TBD    |
| `?test`    | `0x7`/`0b111` | (`racc` & `rcmp`) != 0                        | TBD    |

## Instructions

| Mnemonic          | Arguments             | ID               | Description                                   | Cycles |
|-------------------|-----------------------|------------------|-----------------------------------------------|--------|
| **Data transfer** |                       |        `0b00xxx` | *Data copy*                                   |        |
| `mov`             | `rdst rsrc`           | `0x00`/`0b00000` | 16-bit: Reg -> Reg                            | TBD    |
| `loadw`           | `rdst raddr`          | `0x01`/`0b00001` | 16-bit: Mem -> Reg                            | TBD    |
| `storew`          | `rsrc raddr`          | `0x02`/`0b00010` | 16-bit: Reg -> Mem                            | TBD    |
| `loadb`           | `rdst raddr`          | `0x03`/`0b00011` | 8-bit: Reg -> Mem                             | TBD    |
| `storeb`          | `rsrc raddr`          | `0x04`/`0b00100` | 8-bit: Mem -> Reg                             | TBD    |
| `loadi`           | `imm`                 | `0x07`/`0b00111` | 16-bit: Immediate -> `racc`                   | TBD    |
| **Arithmetic**\*  |                       |        `0b01xxx` | *Two operand arithmetic*                      |        |
| `add`             | `ra rb`               | `0x08`/`0b01000` | Integer addition                              | TBD    |
| `sub`             | `ra rb`               | `0x09`/`0b01001` | Integer subtraction                           | TBD    |
| `and`             | `ra rb`               | `0x0A`/`0b01011` | Bitwise AND                                   | TBD    |
| `or`              | `ra rb`               | `0x0B`/`0b01100` | Bitwise OR                                    | TBD    |
| `xor`             | `ra rb`               | `0x0C`/`0b01101` | Bitwise XOR                                   | TBD    |
| `shl`             | `ra roff`             | `0x0D`/`0b01110` | Bitwise left shift                            | TBD    |
| `shr`             | `ra roff`             | `0x0E`/`0b01111` | Bitwise right shift                           | TBD    |
| **Jump**          |                       |        `0b10xxx` | *Control flow disruption*                     |        |
| `jpi`             | `iaddr`               | `0x18`/`0b10000` | Jump to immediate address                     | TBD    |
| `jp`              | `raddr`               | `0x19`/`0b10001` | Jump to register value address                | TBD    |
| `ret`             |                       | `0x1A`/`0b10010` | Return from function                          | TBD    |
| `calli`           | `iaddr`               | `0x1B`/`0b10011` | Call to immediate address function            | TBD    |
| `call`            | `raddr`               | `0x1C`/`0b10100` | Call to register value address function       | TBD    |
| **Interrupts**    |                       |        `0b11xxx` | *CPU interrupts*                              |        |
| `hlt`             |                       | `0x18`/`0b11000` | Halt CPU until interrupt                      | TBD    |
| `int`             |                       | `0x19`/`0b11001` | Raise a CPU exception                         | TBD    |

\*: Arithmetic operation results are stored in the accumulator register.

### Data transfer

#### `mov rdst radd`

Copies register `rsrc` into register `rdst`.

_Note: Opcode `0x000000` is `mov rfl rfl`, which is effectively a no-op._

```cpp
rdst = rsrc;
```

#### `loadw rdst raddr`

Copies memory at address `raddr` to the 8 lower bits of `rdst`.  
Copies memory at address `raddr + 1` to the 8 higher bits of `rdst`.

```cpp
rdst = (memory[raddr]) & (memory[raddr + 1] << 8);
```

#### `storew rsrc raddr`

Copies the 8 lower bits of `rsrc` to memory at address `raddr`.  
Copies the 8 higher bits of `rsrc` to memory at address `raddr + 1`.

```cpp
memory[raddr] = rsrc & 0x00FF;
memory[raddr + 1] = rsrc >> 8;
```

#### `loadb rdst raddr`

Copies memory at address `raddr` to the 8 lower bits of `rdst`, leaving the 8 upper bits of `rdst` untouched.

```cpp
rdst = (rdst & 0xFF00) & memory[raddr];
```

#### `storeb rsrc raddr`

Copies the 8 lower bits of `rsrc` to memory at address `raddr`, ignoring the 8 upper bits of `rsrc`.

```cpp
memory[raddr] = (rsrc & 0x00FF);
```

#### `loadi imm`

Copies the immediate into `racc`.

```cpp
racc = imm;
```

### Arithmetic

#### `add ra rb`

Adds `rb` with `ra` and store the result to `racc`.

```cpp
racc = ra + rb;
```

Example:

```
loadi 40
mov r7 racc

loadi 2

add racc r7
# now racc = 42
```

#### `sub ra rb`

Subtracts `rb` from `ra` and store the result to `racc`

```cpp
racc = ra - rb;
```

#### `and ra rb`

Stores the result of bitwise AND between `ra` and `rb` to `racc`

```cpp
racc = ra & rb;
```

Example:

```
loadi 0b1100
mov r7 racc

loadi 0b0110

and racc r7
# now racc = 0b0100
```

#### `or ra rb`

#### `xor ra rb`

#### `shl ra roff`

#### `shr ra roff`

Performs a bitshift of `ra` to the right by `roff` bits.  
Bits created on the left (MSBs) are zeroed out.

```cpp
racc = ra >> roff;
```

Example:

```
loadi 0b1000
mov r7 racc

loadi 2

shr r7 racc
# now racc = 0b0010
```

### Jumps

#### `jpi iaddr`

Sets the program counter for the next opcode to `iaddr`, effectively "jumping" to the opcode refered to by this instruction.

```cpp
pc = iaddr;
```

Example:

```
# Decrement variable
loadi 1
mov r7 racc

# Iteration count + 1
loadi 51

.label loop_begin

# Do something here, which will be executed 50 times in total

sub racc r7

# Loop until racc == 0
?nz jpi loop_begin
```

#### `jp raddr`

Sets the program couner for the next opcode to `raddr`.

This may be useful in some situations where you can't use `jpi`, such as if you want to store callbacks or have to jump through a jump table (e.g. `switch` in C++).

```cpp
pc = raddr;
```

Example:

```
loadi jumptable

# Assuming you have a value between 0 and 2 stored in r7

# racc = r7 * 2, since these are words
add racc, r7
add racc, r7

# get the entry from the jump table
loadw racc

# jump to the entry of the jump table, one of l1, l2 or l3
jp racc

.label jumptable
	.word l1
	.word l2
	.word l3

.label l1
	# do something, if r7 == 0

.label l2
	# do something else, if r7 == 1

.label l3
	# do yet another thing, if r7 == 2
```

### `ret`

Returns from a subroutine.

This reads the stored return target into the program counter from the stack then decrements the stack pointer by two.

```cpp
pc = (memory[rsp]) | (memory[rsp + 1] << 8);
rsp -= 2;
```

Example: See `calli`.

### `calli iaddr`

Calls a subroutine.

This increments the stack pointer by two and stores the program counter of the following instruction to the stack, before jumping to `iaddr`.

You may want to follow the ntlabi for subroutine calls, in particular for functions anyhow interfaced with the outside. We do not really care for the purpose of this example.

```cpp
rsp += 2;
memory[rsp] = (pc + 3) & 0x00FF;
memory[rsp + 1] = (pc + 3) >> 8;
pc = iaddr;
```

Example:

```
loadi 5
calli mulbytwo
# racc = 10

.label mulbytwo
add racc, racc
ret
```

### `call raddr`

Similar to what `jp` is to `jpi`, `call` calls a subroutine like `calli` but with an address determined by a register rather than an immediate.

This increments the stack pointer by two and stores the program counter of the following instruction to the stack, before jumping to `raddr`.

```cpp
rsp += 2;
memory[rsp] = (pc + 3) & 0x00FF;
memory[rsp + 1] = (pc + 3) >> 8;
pc = raddr;
```

### `hlt`

Halts the CPU until an interrupt is raised by any of the interrupt-enabled peripherials.

If `_INTON` is disabled, the implementation may be able to power off automatically.
If `_INTLOCK` is set while `_INTON` is unset, the implementation behavior will rather be to reset the CPU.
However, it is not required care as to whether `_INTxON` are set. If `_INTON` is set, but none of the interrupts are enabled, `hlt` will by definition hang the CPU until its power is cut off.

When woken up by an interrupt in a halted state, the ISR will be called as usual.

See [IO and interrupts](io.md)

### `int`

Manually generate a CPU exception with ID `_USER` (`0`), as if it was generated by the current opcode.

See [IO and interrupts](io.md)
