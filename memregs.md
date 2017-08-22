## Memory and registers

The 16 registers are __16-bit__ wide. Memory addressing is __16-bit__, and the memory atom type is __16-bit__, effectively enabling 128KiB memory. On the bootup, every register defaults to `0`.

ntl is a __load-store__ architecture, which means that memory access is done through special instructions. It is a __modified Harvard__ architecture, so the scratchpad memory and the program memory are separate, but can be accessed through separate instructions.

#### Register cheatsheet

| Name   | ID    | Description           |
|--------|-------|-----------------------|
| `rfl`  | `0x0` | __FL__ag register     |
| `ridt` | `0x1` | __IDT__ address       |
| `racc` | `0x2` | __ACC__umulator       |
| `rsp`  | `0x3` | __S__tack __P__ointer |

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
