# Egg Language Specification

## Overview

Egg is a very simple and flexible transpilation language designed as a thin layer between high-level concepts and assembly code. It transpiles to assembly for multiple architectures (x86, x86-64, ARM, ARM64) and is designed to give full control over the final code while maintaining simplicity for common operations like function calls, syscalls, loops, and conditionals.

### Design Philosophy

- **Simplicity First**: The parser is extremely simple and lightweight
- **Flexible over Strict**: The language intentionally allows generating "wrong" code if the user wants that - it's a feature, not a bug
- **Thin Abstraction Layer**: Minimal overhead between Egg and final assembly
- **Multi-architecture**: Single source compiles to different ISAs
- **Integration with r2**: Reuses radare2's ecosystem (assembler, syscalls, analysis)

## Language Syntax

### Basic Structure

Egg uses a simple syntax inspired by C but much more restrictive. The parsing is character-by-character and context-based.

```egg
// Function definition
function_name@type(stackframe, staticframe) {
    // body
}

// Comments
/* block comment */
// line comment  
# line comment
```

### Function Types and Signatures

#### Syntax: `name@type(stackframe, staticframe) { body }`

- **name**: Function identifier
- **type**: One of the function types below
- **stackframe**: Stack space for local variables (bytes)
- **staticframe**: Stack space for static data like strings (bytes)

#### Function Types

- **`global`**: Global function symbol (main entry point)
- **`inline`**: Function is inlined when called
- **`syscall`**: Define syscall calling convention
- **`alias`**: Create alias (becomes `equ` in assembly)
- **`data`**: Data section definition
- **`naked`**: Raw label without prologue/epilogue

Example:
```egg
main@global(128, 64) {  // 128 bytes for locals, 64 for static data
    // function body
}
```

### Variables

#### Special Variables

- **`.arg`** / **`.arg0`**, **`.arg1`**, etc: Function arguments
- **`.var0`**, **`.var1`**, etc: Local variables (stack-based)
- **`.fix0`**, **`.fix1`**, etc: Static variables (in static frame)
- **`.ret`**: Return value register (eax/rax for x86, r0 for ARM)
- **`.bp`**: Base pointer register
- **`.sp`**: Stack pointer register  
- **`.pc`**: Program counter register

#### Usage
```egg
.var0 = 5;
.var1 = .arg0 + .arg1;
.ret = .var0;
```

### Syscalls

Two ways to define syscalls:

#### 1. Simple Definition
```egg
write@syscall(4);
exit@syscall(1);
```

#### 2. Custom Syscall Body
```egg
exit@syscall(1);
@syscall() {
    : mov eax, `.arg`
    : int 0x80
}
```

### Control Flow

#### Conditionals
```egg
if (.var0 == 0) {
    // true block
} else {
    // false block  // Not fully implemented yet
}
```

#### Loops
```egg
.var0 = 10;
while(.var0) {
    // loop body
    .var0 -= 1;
}
```

#### Jumps and Labels
```egg
goto(label_name);

:label_name:
// continue here
```

#### Break
```egg
break();  // Emits trap instruction (int3 on x86)
break;    // Same effect
```

### Strings and Data

#### String Literals
```egg
// Double quotes = filtered (escape sequences processed)
write(1, "Hello\nWorld\n", 12);

// Single quotes = unfiltered  
write(1, 'Hello\nWorld\n', 12);
```

#### Data Sections
```egg
mydata@data(4) {
    "Hello World\0"
}
```

### Inline Assembly

Lines prefixed with `:` are passed directly to the assembler:
```egg
: mov eax, 0x80
: int 0x80  
: jmp 0x8049000
```

### Math Operations

#### Assignment and Math
```egg
.var0 = 42;
.var1 = .var0 + 10;
.var2 = .arg0 * 2;
```

#### Supported Operators
- `+` - Addition  
- `-` - Subtraction
- `*` - Multiplication
- `/` - Division
- `&` - Bitwise AND
- `|` - Bitwise OR  
- `^` - Bitwise XOR

#### Operator Precedence
1. `* /` (highest)
2. `+ -`
3. `& | ^` (lowest)

## Current Architecture Support

### Supported Architectures
- **x86** - 32-bit
- **x86-64** - 64-bit  
- **ARM** - 32-bit
- **ARM64** - 64-bit
- **ESIL** - Environmental Intermediate Language (for analysis)
- **trace** - Call tracing backend

### Platform Support
Egg detects platform and adjusts syscall conventions:
- **Linux**: Standard Linux syscalls
- **macOS/Darwin**: macOS syscall conventions
- **Windows**: Basic support (limited)

## Parsing and Compilation Process

### One-Pass Compiler
Egg is a **one-pass compiler**, which means:
- Functions must be defined before being called
- Stack frame size must be defined at function start
- Forward references are limited

### Parser States
The parser maintains several context states:
- Normal mode
- String parsing mode  
- Comment mode
- Various function definition modes (syscall, data, inline, etc.)

### Compilation Pipeline
1. **Source Parsing**: Character-by-character parsing
2. **Tree Building**: Simple AST-like structure
3. **Code Generation**: Architecture-specific emit backends
4. **Assembly**: Uses r_asm for final assembly
5. **Binary**: Optional binary generation

## Integration with Radare2

### Reused Components
- **r_asm**: Assembly and disassembly
- **r_syscall**: Syscall definitions and numbers
- **r_anal**: Analysis integration (future)
- **r_bin**: Binary format generation

### Plugin System
Egg supports plugins for:
- Shellcode generators
- Encoders
- Custom backends

### Configuration
- Environment variables for include paths
- SDB database for options
- Command line parameters via ragg2

## Current Status and Limitations

### What Works Well
- Basic function definitions and calls
- Syscalls with automatic setup
- Simple conditionals and loops  
- String handling with escape sequences
- Multi-architecture code generation
- Inline assembly support
- Variable access and basic math

### Known Limitations
- **Else blocks**: Not fully implemented in conditionals
- **Complex expressions**: Limited math operator support
- **Pointers**: Basic pointer support (*.var, &var) incomplete
- **Calling conventions**: Only basic cdecl-style syscall calling
- **Error handling**: Limited error reporting
- **Type system**: No type checking or validation
- **Memory management**: Manual stack management only

### Architecture-Specific Issues
- **ARM**: Register mapping issues, limited testing
- **64-bit**: Endian and size handling problems in some cases
- **Position Independent Code**: Limited support for PIC requirements

## File Formats and Extensions

### Source Files
Egg files typically use `.r` extension (from ragg2 tool)

### Example Program
```egg
#!/usr/bin/ragg2 -X
goto(main);

write@syscall(4);
exit@syscall(1);
@syscall() {
    : mov eax, `.arg`
    : int 0x80
}

main@global(128,128) {
    write(1, "Hello Egg!\n", 12);
    exit(0);
}
```

## Future Directions

### Immediate Needs
1. **Complete else implementation** for conditionals
2. **Better error messages** and debugging
3. **Enhanced pointer operations**
4. **More math operations** and complex expressions

### Long-term Goals
1. **Type system** for basic validation
2. **Standard library** of common functions
3. **Variable-length arguments** support
4. **Structure and array support**
5. **Optimization passes** for generated code
6. **IDE integration** and syntax highlighting

### Integration Possibilities
- **Better r2 integration** for live patching
- **WebAssembly backend** for browser use
- **Custom backends** for embedded systems
- **Binary rewriting** tools