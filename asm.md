## Assembly language

#### Notation

Identifiers are case sensitive. An user identifier can begin with any alphanumerical character. Only built-in identifiers, such as flag offsets, may begin by `_`.  
Register names are prefixed by `r`.

#### Instructions

An instruction is defined by writing its mnemonic, followed by a space and an operand list, if required. Operands in the operand list are separated by whitespace characters. Operands can be an immediate value or a register name. For example:

`mov racc r0`

This is read as a `mov` instruction, with the `racc` register as the first operand and `r0` as the second operand. When the amount of arguments in the operand list does not match to what the instruction expects, an error occurs.  
Newlines separate instructions.

#### Immediates

Immediate values can represent any value. Those can be determined directly at compile-time.

Labels will expand into an offset value in the program.  
Integral values will expand appropriately in the program as required. Those can be written in decimal (`1234`), hexadecimal (`0x` prefixed - `0xBEEF`) or binary (`0b` prefixed - `0b1100`).  

#### Comments

Comments are pieces of text that are ignored by the assembler and not parsed in any way. Comments can begin anywhere in a line from the `#` character and finishes at the end of this line.

    # this is a comment
    # this is another comment
    mov racc, r0 # explain something complicated
    mov racc, r1
    # mov racc, r2 - commented instructions are ignored

#### Assembler directives

Assembler directives allows using assembly or assembler related features not available otherwise. These are prefixed by `.`. For example, the `.at` directive forces the following instruction to be located at that specific area.

| Name       | Description                                   |
|------------|-----------------------------------------------|
| `.at`      | Force the assembler location counter.         |
| `.data`    | Inline an immediate value within the program. |
| `.include` | Include a file within this file.              |
| `.label`   | Define a reference to the next word.          |

- ##### `.at offset`

Forces the assembler location counter.  
In other words, the next instruction (or assembler directive) will be located at the given offset in the resulting flat binary.

    .at 0x1000
    mov racc, r0
    mov racc, r1

This piece of code will have the `mov racc, r0` instruction located at `0x1000` in the flat binary. Do remember that the memory word is 16-bit in ntl, so the actual byte offset would be `0x2000`.  
The instruction that follows naturally is located at `0x1002`.

- ##### `.data size data`

Inline an immediate value within the program binary. Extra bits will be filled with zero.

	.data 8 0xDEADBEEF
	.data 11 "hello world"

This example will hold, in program memory:

	BEEF DEAD 0000 0000 6865 6C6C 6F20 776F 726C 0064

- ##### `.include`

Includes a file.  
Including a file copies its content at the position of the `.include` directive in the current file.

Example:

`b.nts` file:

	.label someFunction
		add rg0, rg1
		ret

`a.nts` file:

	.include "b.nts"
	.label main
		ldi rg0, 8
		ldi rg1, 2
		calli someFunction

- ##### `.label`

Defines a label, that is, a macro variable to the next encountered word.  

	.label infiniteLoop
		jmpi infiniteLoop
		
#### Warnings and errors
