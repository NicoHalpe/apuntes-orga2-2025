# Assembly Programming Reference Guide

## Register Architecture

### x86-64 Register Notation
- **R__** → 64 bits (8 bytes)
- **E__** → 32 bits (4 bytes)
- **_X** → 16 bits (2 bytes)
- **_H** → 8 bits (1 bytes) - Most Significant
- **_L** → 8 bits (1 bytes) - Least Significant

## Data Types and Sizes

| Type | Size | Bytes |
|------|------|-------|
| BYTE | 8 bits | 1 |
| WORD | 16 bits | 2 |
| INTEGER | 32 bits | 4 |
| FLOAT | 32 bits | 4 |
| LONG | 64 bits | 8 |
| DOUBLE | 64 bits | 8 |
| CHAR | 8 bits | 1 |
| BOOL | 8 bits | 1 |
| POINTERS | 64 bits | 8 |

## Register Categories

### Non-Volatile Registers (BY CPU SIZE)
| **64-bit** | **32-bit** |
|------------|------------|
| RBX | EBX |
| RBP | EBP |
| R12 | ESI |
| R13 | EDI |
| R14 |
| R15 |

### Volatile Registers

|**64-bit** | **32-bit** | **16-bit** | **8-bit High** | **8-bit Low** |
|-----------|------------|------------|----------------|---------------|
| RAX | EAX | AX | AH | AL |
| RCX | ECX | CX | CH | CL |
| RDX | EDX | DX | DH | DL |
| RSI | ESI | SI |  | SIL |
| RDI | EDI | DI |  | DIL |
| RSP | ESP | SP |  | SPL |
| R8 | R8D | R8W |  | R8B |
| R9 | R9D | R9W |  | R9B |
| R10 | R10D | R10W |  | R10B |
| R11 | R11D | R11W |  | R11B |

## Function Calling Convention

### Parameter Passing
- **Integers and Pointers:** RDI, RSI, RDX, RCX, R8, R9 (left to right)
- **Floating Point:** XMM0, XMM1, ..., XMM7 (left to right)
- **Overflow:** Stack (right to left push order)

### Return Values
- **32-bit:** EAX
- **64-bit:** RAX
- **Floating Point:** XMM0

### Special Notes
- RSP/ESP can be considered non-volatile due to its stack pointer function
- R10 and R11 are neither parameter registers nor non-volatile
- XMM8 to XMM15 are available but not used for parameter passing

## Stack Management

### 16-byte Aligned Stack (64-bit)

#### Stack Operations
- **PUSH:** Decrements RSP by 8, then writes value
- **POP:** Reads value, then increments RSP by 8
- **CALL:** Pushes return address, jumps to function
- **RET:** Pops return address, returns to caller

#### Stack Order (LIFO)
```asm
; Save order:
push rax    ; 1. Save EAX
push rbx    ; 2. Save EBX

; Restore order (reverse):
pop rbx     ; 1. Restore EBX
pop rax     ; 2. Restore EAX
```

### Function Prologue/Epilogue

#### Prologue
```asm
push rbp
mov rbp, rsp
push r12
push r13
push r14
push r15
```

#### Epilogue
```asm
pop r15
pop r14
pop r13
pop r12
pop rbp
ret
```

## Type Conversions

### Integer to Float/Double
```asm
cvtsi2ss    ; Int → Float (32-bit precision)
cvtsi2sd    ; Int → Double (64-bit precision)
```

### Double to Integer
```asm
cvtsd2si    ; Double → Int (rounds to nearest, 0.5 rounds up)
cvttsd2si   ; Double → Int (truncates, rounds toward zero)
```

### Float Conversions
```asm
cvtss2sd    ; Float → Double
cvtss2si    ; Float → Int (rounds to nearest)
cvttss2si   ; Float → Int (truncates)
```

## Packed vs Scalar Operations

- **P (Packed):** Multiple operations on same register with different values
- **S (Scalar):** Operations on least significant value only

## Arithmetic Operations

### Integer Multiplication
```asm
mul     ; Unsigned multiplication (no immediates) (reg_dest, reg_mul) or (reg_mul) -> rax
imul    ; Signed multiplication (allows immediates)
```

### Floating Point Multiplication
```asm
mulss   ; Float multiplication (unsigned)
mulsd   ; Double multiplication (unsigned)
```

### Utility Operations
```asm
inc reg         ; Increment register by 1
dec reg         ; Decrement register by 1

lea             ; Load Effective Address (given a register and a adress it saves the address direction into the register)
[rel direction] ; Uses the relative adress to give information (usefull when you set hard-code strings into your code)

shl             ; Shift Left Unsigned
shr             ; Shift Rigth Unsigned
```

## Control Flow

### Comparison and Testing
```asm
cmp op1, op2    ; Compare operands (sets flags)
test op1, op2   ; Bitwise AND (sets flags, doesn't store result)
```

### Conditional Jumps
| Instruction | Condition | Description |
|-------------|-----------|-------------|
| JE/JZ | == / Zero flag set | Jump if equal/zero |
| JNE/JNZ | != / Zero flag clear | Jump if not equal/not zero |
| JL | < | Jump if less than |
| JLE | <= | Jump if less or equal |
| JG | > | Jump if greater |
| JGE | >= | Jump if greater or equal |
| JA | > (unsigned) | Jump if above |
| JAE | >= (unsigned) | Jump if above or equal |
| JC | Carry flag set | Jump if carry |
| JMP | Always | Unconditional jump |

## Assembly Directives

### Definitions
```asm
%define ALUMNO 10       ; Define symbolic constant
ETIQUETA EQU NUMERO     ; Alternative syntax

section .data
    CLT: db "CLT", 0    ; Define strings
    RBO: db "RBO", 0
```

## C Library Headers

### Common Headers
```c
#include <stdio.h>      // I/O functions: printf, fprintf, fopen, fclose
#include <stdlib.h>     // Standard library: malloc, free, atoi, exit
#include <stdint.h>     // Fixed-width integer types: int8_t, uint64_t
#include <ctype.h>      // Character functions: isdigit, tolower
#include <string.h>     // String functions: strlen, strcpy, memcmp
#include <math.h>       // Math functions: sin, cos, sqrt, log
#include <stdbool.h>    // Boolean types: true, false
#include <unistd.h>     // POSIX constants: STDIN_FILENO, NULL
#include <assert.h>     // Assertion macro: assert()
#include "structs.h"    // Custom data structures
```

### Key Functions
- **fprintf:** Like printf, but writes to file stream
- **malloc:** Allocate memory  (uint_t size of bytes)
- **free:** Deallocate memory  (*pointer)
- **strcpy:** Copy string (destination first, then source) (*char1, *char2)
- **strcmp:** Compare two strings lexicographically, if 0, they are equal (*char1, *char2)

## GDB Debugging

### Basic Commands
```bash
gdb <executable>    # Start GDB with executable
r                   # Run program
n                   # Next line
q                   # Quit GDB
```

### Breakpoints
```bash
b <function_name>               # Break at function
b <filename>:<line_number>      # Break at specific line
```

### Register Inspection
```bash
p $rcx                          # Print RCX register content
p *(uint32_t*)$rbp              # Cast and dereference RBP
p {struct_name}$reg             # Print the structure in correct format
p ($reg + offset)               # Print the reg + an offset (any operation should be valid but I wont promise this)
p {struct_name} $reg @N         # Print N structs
```
### Memory Examination
```bash
x /<address_expression>                    # Examine memory
x /<format> <address_expression>           # With format
x /<length><format> <address_expression>   # With length and format
```

#### Format Modifiers
| Modifier | Description |
|----------|-------------|
| o | Octal |
| x | Hexadecimal |
| d | Decimal |
| u | Unsigned decimal |
| t | Binary |
| f | Floating point |
| a | Address |
| c | Character |
| s | String |
| i | Instruction |

#### Size Modifiers
| Modifier | Size | Description |
|----------|------|-------------|
| b | 8-bit | Byte |
| h | 16-bit | Half word |
| w | 32-bit | Word |
| g | 64-bit | Giant word |

## Best Practices

1. **Stack Management:** Always maintain 16-byte alignment
2. **Register Usage:** Preserve non-volatile registers in functions
3. **Parameter Passing:** Follow x86-64 calling convention
4. **Memory Management:** Free all allocated memory
5. **Debugging:** Use GDB effectively for troubleshooting

---

*This reference covers essential x86-64 assembly programming concepts, calling conventions, and debugging techniques.*