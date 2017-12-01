---
title: I/O and interrupts
---

I/O and interrupts
=====

#### Interrupts

The CPU can be woken up by __8__ interrupts. An interrupt stops the operation of the CPU and can be triggered either by the user, either by a peripherial.  
They can be enabled globally through the `_INTON` flag and enabled individually through the `_INTxON` flags.

The `ridt` register is the offset to the Interrupt Descriptor Table. The IDT is a table with 8 entries of 16 bits. Each entry is an offset matching to an Interrupt Service Routine.

ISRs are routines called when an interrupt is called.  
The return address matches to the instruction that should have been executed when being interrupted. However, there are things that the application programmer needs to be aware of:
- An ISR is a regular, non-returning function.
- Interrupts from ID `1` to `7` do not push any parameter data through the stack.
- Interrupt `0` is special and has extra things to take care of. _See: CPU exceptions_
- Interrupts do not affect registers. They should be saved and restored by the ISR in order to resume the task that was interrupted.
- When an interrupt rises, the `_INTLOCK` flag is set. It should be disabled by the ISR properly, because it temporarily disables interrupts.
- When interrupts are disabled (when `_INTON` and `_INT*ON` are not set), those interrupts are rejected and will never interrupt the CPU, even when reenabling interrupts.
- When interrupts are enabled, but `_INTLOCK` is set, the interrupts will stack in a dedicated interrupt stack. This stack has a limited size, defined by the implementation, so pushing an interrupt to the interrupt stack will effectively reject the interrupt.

##### CPU exceptions (Interrupt `0`)

A CPU exception is a special interrupt that cannot be disabled. Unlike other interrupts, it requires a special ISR to handle the CPU exception ID, which is pushed on the stack as a parameter. When a CPU exception rises while `_INTLOCK` is set or `_INTON` is disabled, the CPU will halt. The following CPU exception IDs exist:

| Name          | ID       | Description               |
|---------------|----------|---------------------------|
| `_ARITHMETIC` | `0x0000` | Arithmetic exception      |
| `_ILLOP`      | `0x0001` | Illegal operation         |
| `_UNIMPL`     | `0x0002` | Incomplete implementation |

#### Memory Mapped I/O

In order to interact with peripherials, ntl implementations may implement Memory Mapped I/O.  
MMIO uses the same address space as memory. However, MMIO regions can be listened to by peripherials.

32KiB of the memory address space is reserved for MMIO, within the `0xC000-0xFFFF` range.  
_Within the MMIO reserved region_, memory read operations are implementation-defined behavior.

##### Reference implementation behavior

The implementation-defined behavior depends on peripherials.
