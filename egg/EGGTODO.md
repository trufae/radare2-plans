# Egg Language TODO List

This document outlines the remaining tasks, improvements, and known issues for the Egg language compiler based on analysis of the current codebase.

## Critical Issues (High Priority)

### Parser and Language Core
- [ ] **Fix else implementation** in conditional statements (#776, #777, #796)
  - Current if-else logic is incomplete and buggy
  - Nested if-else not working properly
  - Else blocks are not properly generated

- [ ] **Memory leak fixes** in language parsing
  - Syscall name/arg arrays not freed (#375)
  - String allocations in `r_egg_mkvar` (#492)
  - General memory management in parser

- [ ] **Stackframe calculation issues** 
  - Delta calculation problematic (#488)
  - Stackfixed offset handling issues (#570)
  - Proper frame management for nested calls

### Architecture Support
- [ ] **ARM64 and ARM backends incomplete**
  - Many unhandled cases (#259, #261)
  - Missing proper register handling
  - Stack management issues (#178, #181)

- [ ] **64-bit support fixes**
  - Endian and size handling problems (#217, #399)
  - Offset adjustment issues (#168)
  - Register mapping improvements needed

## Important Improvements (Medium Priority)

### Enhanced Language Features
- [ ] **Better pointer operations**
  - Implement full pointer dereference (*) (#147)
  - Add address-of operator (&) (#147)
  - Pointer arithmetic support

- [ ] **Expression evaluation improvements**
  - Support != 0 comparisons (#341, #346, etc)
  - Enhanced operator precedence
  - Complex expression parsing (#1014, #1016, #1017)

- [ ] **Variable system enhancements**
  - Size specifiers for variables (b, w, l, q)
  - Signed/unsigned variable support (#414)
  - Variable lifetime management

### Code Quality and Structure
- [ ] **Split large functions**
  - `r_egg_lang_parsechar` too long (#928)
  - Better separation of parsing logic
  - Modularize emit backends

- [ ] **Error handling improvements**
  - Better error messages and location tracking
  - Proper error codes and recovery
  - Input validation

- [ ] **Plugin system completion**
  - Fastcall implementation (#612)
  - Custom backend support
  - Encoder plugin improvements

## Code Generation and Backend Issues

### x86/x86-64 Backend
- [ ] **Fix stack alignment**
  - Add 'andb rsp, 0xf0' for proper alignment (#36)
  - 64-bit frame management improvements
  - Proper prologue/epilogue generation

- [ ] **Register allocation improvements**
  - Better use of available registers
  - Calling convention handling
  - Multi-register return values

- [ ] **Comparison fixes**
  - != 0 comparison support (#341, #346)
  - Signed/unsigned comparison handling
  - Conditional jump optimization

### ARM/ARM64 Backend
- [ ] **Complete register implementation**
  - Proper r0-r7 mapping for args (#201, #240)
  - ARM64 specific register handling
  - Thumb mode support

- [ ] **Syscall conventions**
  - Platform-specific syscall handling
  - ARM64 Linux vs macOS differences
  - Proper svc instruction usage

### ESIL Backend
- [ ] **Complete ESIL integration**
  - Better ESIL expression generation
  - Analysis improvements
  - Symbolic execution support

## Standard Library and Ecosystem

### Core Library Development
- [ ] **Standard library functions**
  - String operations (strlen, strcpy, etc)
  - Memory operations (memcpy, memset)
  - Math functions
  - File I/O wrappers

- [ ] **Include system**
  - Preprocessor support (#47 from README)
  - File inclusion with search paths
  - Conditional compilation support

- [ ] **Testing framework**
  - More comprehensive test suite
  - Regression tests for each architecture
  - Performance benchmarks

### Tool Integration
- [ ] **Better r2 integration**
  - Live patching tools
  - Visualization features
  - Analysis feedback loop

- [ **Development tools**
  - Syntax highlighting definitions
  - Linter and validator
  - Debugging support
  - IDE plugins

## Future Enhancements (Low Priority)

### Language Extensions
- [ ] **Type system foundation**
  - Basic type checking
  - Structure support
  - Array types and indexing
  - Enum definitions

- [ ] **Advanced control flow**
  - Switch/case statements
  - Break/continue for loops
  - Function return statements
  - Exception handling basics

- [ ] **Macro system**
  - Preprocessor macros
  - Code generation macros
  - Conditional compilation

### Optimization and Performance
- [ ] **Code optimization passes**
  - Dead code elimination
  - Common subexpression elimination
  - Loop optimizations
  - Register allocation improvements

- [ ] **Position Independent Code (PIC)**
  - Better PIC support across architectures
  - Relative addressing improvements
  - Shared library generation

- [ ] **Size optimization**
  - Smaller code generation
  - Code deduplication
  - Better string handling

### New Backends and Targets
- [ ] **WebAssembly backend**
  - WASM generation for browser use
  - JavaScript target
  - Node.js integration

- [ **Microcontroller targets**
  - AVR backend
  - RISC-V support
  - Embedded optimization options

- [ ] **Binary rewriting tools**
  - Patch generation using Egg
  - Binary transformation
  - Decompilation integration

## Documentation and Community

### Documentation
- [ ] **Complete API documentation**
  - Function reference
  - Architecture-specific notes
  - Migration guides

- [ ] **Tutorial and examples**
  - Getting started guide
  - Cookbook of common patterns
  - Advanced usage examples

- [ **Developer documentation**
  - Parser internals guide
  - Backend development guide
  - Plugin development tutorial

### Community Infrastructure
- [ ] **Bug triage and tracking**
  - Known issues documentation
  - Bug report templates
  - Regression testing framework

- [ ] **Contribution guidelines**
  - Code style guidelines
  - Testing requirements
  - Pull request templates

## Technical Debt Cleanup

### Code Modernization
- [ ] **Replace hardcoded limits**
  - 256 function limits for aliases, syscalls, inlines
  - 32-level nested context limits
  - 1024 character element limits

- [ **Improve const correctness**
  - Add const qualifiers where appropriate
  - Better function signatures
  - Read-only parameter improvements

- [ **Remove deprecated code**
  - Clean up commented-out code blocks
  - Remove unused functions
  - Simplify conditional compilation

### Build and Dependencies
- [ ] **Meson build improvements**
  - Better dependency handling
  - Cross-compilation support
  - Packaging improvements

- [ **Testing infrastructure**
  - Automated testing pipeline
  - Architecture-specific test matrices
  - Performance regression testing

## Research and Experimental Features

### Advanced Features
- [ **JIT compilation integration**
  - Runtime compilation support
  - Dynamic code generation
  - Self-modifying code support

- [ **Security features**
  - Control flow integrity checks
  - Stack canary integration
  - ASLR-friendly code generation

- [ **Binary diff/patch**
  - Egg-based binary diffing
  - Automated patch generation
  - Version control integration

### Analysis and Verification
- [ **Static analysis**
  - Type checking implementation
  - Dead code analysis
  - Security vulnerability detection

- [ **Formal verification**
  - Behavior verification
  - Correctness proofs
  - Property checking

---

**Priority Legend:**
- 🔴 Critical (blocker issues, crashes, major functionality)
- 🟡 Important (significant improvements, missing features)
- 🟢 Nice to have (enhancements, future work)

**Note**: This TODO is based on TODO/XXX/FIXME comments found in the codebase and architectural analysis. Priorities may shift based on user needs and development resources.