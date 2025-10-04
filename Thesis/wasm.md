# Wasm (Web Assembly)

These are general notes I wrote/am writing to get familiar with Web Assembly. Notes are based off the (WebAssembly Specification)[https://www.w3.org/TR/wasm-core-1/]

### Overview

WebAssembly is a open specification published by the World Wide Web Consortium. WebAssembly is a "save, portable, low-level code format designed for efficient execution and compact representation. Its main goal is to enable high performance applications on the Web, but it does not make any Web-specific assumptions or provide Web-specific features, so it can be employed in other environments as well."

WebAssembly (Wasn from here on) can be a target to compile to for many languages, enabling compiled languages like C, C++ etc. to be ran in a browser environment with near native performance (which isn't actually entirely true, inc sources).

### Concepts

- **Values**:
    - Wasm has 4 basic value types: integers & numbers (32 and 64 bit width)
    - Booleans are represented as 32 bit integers (what a waste)
    - Memory addresses are 32 bit integers

 - **Instructions**:
     - Wasm's computational model is based on a stack machine
     - Code is comprised of a sequence of instructions that are executed in order
     - Instructions manipulate values on an implicit operand stack
     - Types (2):
         - Simple: Basic operands on data, pop args from stack, push results back
         - Control: Alters control flow
     - Control flow is structured, expressed with well-nested constructs such as blocks, loops, conditionals
     - Branches can only target such constructs (no jmp to arbitrary instruction address like ASM)

- **Traps**:
    - Traps are when the execution machine enters a state where it can no longer continue
    - This immediately aborts execution
    - Wasm code can't handle traps, but they are reported to the outside environment which can handle them accordingly

- **Functions**:
    - Code is organized into seperate functions
    - Each function takes a sequence of values as parameters and returns a sequence of values as a result (only one res value in Wasm 1.0 spec)
    - Functions can call each other, even recursively
    - This results in an implicit call stack that can't be directly accessed
    - Functions can declare mutable local variables that can function as virtual registers

- **Tables**:
    - A table is an array of opaque values of a particular element type
    - Programs can select values indirectly through a dynamic index operand
    - The only available element type is and untyped function reference (wasm 1.0)
    - This means a program can call functions indirectly through indexing into a table, emulating function pointers

- **Linear Memory**:
    - Linear memory is created (allocated) with an initial size, but can be grown dynamically
    - A program can load/store values from/to a linear memory at any address (including unaligned!)
    - Integer load/stores can specify a storage size which is smaller than the size of the respective value type
    - Trap occurs upon an out of bounds access attempt

- **Modules**:
    - A Wasm binary takes the form of a module that contains definitions for functions, tables, linear memories and immutable or mutable global variables
    - Definitions can be imported by specifying a module/name pair and a suitable type
    - Definitions can be exported under one or more names
    - Modules can define initialization data for their memories or tables that take the form of segments copied to given offsets
    - Modules can define a start function that is automatically executed

- **Embedder**:
    - A Wasm implementation is typically embedded into a host environment (web browser).
    - This env defines how loading is initiated, how imports are provided, how exports can be accessed

### Semantic phases

- **Decoding**:
    - Wasm modules are distributed in a binary format (rather proprietary)
    - Decoding processes the format and converts it into an internal representation of a module (e.g. compiled to machine code)
    - **In the specification, the binary format is modelled by abstract syntax, in these notes this will be replaced with a textual representation (.wat and/or .asm where applicable)**

- **Validation**:
    - Decoded modules are checked for validity to guarantee the module is meaningful and safe.
    - This comprises of type checking of functions and the instruction sequences in their body

- **Execution**:
    - Instantiation:
        - A module instance is the dynamic representation of a module, with its own state and execution stack
        - Instantiation executes the module body, given definition for all its imports
        - Istantiation initializes globals, memories and tables and invokes the modules start function if defined
        - It returns the instances of the module's exports

    - Invocation:
        - Once instantiated, further Wasm computations can be initiated by invoking an exported function on a module instance

Instantiation and invocation are operations within the embedding environment.


## Structure

As noted before, the official specification defines the Wasm representation in abstract syntax. For easier comprehension, I will (attempt to) replace these representations with equivalent text based forms, either in the .wat Wasm text format or in 32 bit x86 assembly.

### Values

Wasm operates on primitive numeric values. Immutable sequences can occur to represent more complex data, such as strings or other vectors.

#### Bytes

The simplest form of value are raw uninterpreted bytes

```wat
;; Examples
0x00
0x01
0xFF

;; Interpreted as i32 constants
i32.const 0 	;; 0x00
i32.const 255	;; 0xFF
```

#### Integers 

Integers are distinguished by their bit width and signedness. Uninterpreted integers are integers that are stored as unsigned, but operations can treat them as signed (two's complement).

```wat
;; Common integer types
u32, u64 s32, s64, i8, i16, i32, i64

;; Only i32 and i64 are real, rest is interpreted by operation

;; Unsigned vs signed disctinction is in operations
i32.const 0 					;; 0u32
i32.const 4294967295 			;; 0xFFFFFFFF, still stored as i32
i32.const -1					;; -1 in two's complement

;; Interpretation depends on instruction
i32.div_s						;; signed division (s32)
i32.div_u						;; unsigned division (u32)
i32.lt_s						;; signed comparison
i32.lt_u						;; unsigned comparison

i32.lt_s 0xFFFFFFFF i32.const 1 ;; signed comparison: -1 < 1 -> true
i32.lt_u 0xFFFFFFFF i32.const 1 ;; unsigned comparison: 4294967295 < 1 -> false

;; i8/i16 are promoted to i32 on load/store
i32.load8_s						;; load signed 8-bit from memory, sign-extend to i32
i32.load8_u 					;; load unsigned 8-bit, zero-extend to i32
i32.load16_s					;; load signed 16-bit, sign-extend
i32.load16_u					;; load unsigned 16-bit, zero-extend
```

#### Floating-point numbers

Wasm floating point numbers correspond to the IEEE-754 binary32 and binary64 formats. Only `f32` and `f64` floating-pont types exist.

- Components:
    - Sign: + or -
    - Exponent: e
    - Mantissa: m
    
- Normal numbers:
    - (1 + m * 2^-M) * 2^e
    - M: number of significant bits (f32: 23, f64: 52)
    - Leading bit of normal numbers = 1
    
- Subnormal numbers:
    - Exponent = minimum possible
    - Leading significant bit = 0
    - Represent very small numbers close to 0

- Special values:
    - Zeros: `+0`, `-0`
    - Infinities: `+inf`, `-inf`
    - NaN (Not-a-Number):
        - Has a payload in the mantissa
        - Canonical NaN -> MSB of payload = 1, others = 0
        - Arithmetic NaN -> MSB of payload = 1, others arbitrary
    - No distinction between signaling and quiet NaNs in Wasm

```wat
f32.const 0.0 		;; +0
f32.const -0.0 		;; -0
f32.const 1.5 		;; normal number
f32.const 1e-40 	;; subnormal number
f32.const inf 		;; positive infinity
f32.const -inf 		;; negative infinity
f32.const nan 		;; canonical NaN

;; Operations
f32.add
f32.sub
f32.mul
f32.div
f32.sqrt
```

#### Names

Names in Wasm are a sequence of characters, characters are Unicode scalar values. Names are stored in UTF-8. The length of a name is bounded by the length of its UTF-8 encoding, not the number of characters

```wat
;; Names appear for functions, globals, locals, modules, etc.

(module
  (func $my_function (param $x i32) (result i32)
    local.get $x
  )
)

;; $my_function is a name
```

Names must be valid UTF-8 and encode characters from the allowed Unicode ranges.

```
;; Internally, Wasm stores names as UTF-8 bytes

$my_function -> [0x6D,0x79,0x5F,0x66,0x75,0x6E,0x63,0x74,0x69,0x6F,0x6E]

```

### Types

Various entities in Wasm are classified by types. Types are checked during validation, instantiation and possibly execution.

- **Value Types**:
    - i32, i64, f32, f64

- **Result Types**:
    - Classify the result of executing instructions or blocks

- **Function Types**:
    - Classify the signature of functions

- **Limits**: 
    - Classify the size range of resizeable storage associated with memory types and table types

- **Memory Types**:
    - Classify linear memories and their size range
    - Limits constrain the minimum and optionally maximum size of a memory. The limits are given in units of page size.

- **Table Types**:
    - Classify tables over elements of element types within a size range
    - Limits constrain the minimum and optionally maximum size of a table. The limits are given in numers of entries.

- **Global Types**:
    - Classify global variables, which hold a value and can either be mutable or immutable

- **External Types**:
    - Classify imports and external values with their respective types

### Instructions

Wasm consists of sequences of instructions. Its computational model is based on a stack machine. Instructions manipulate values on an implicit operand stack, popping argument values and pushing result values.

In addition to dynamic operands from the stack, some instructions have static immediate arguments, which are part of the instruction itself.

#### Numeric Instructions

Numeric instructions operate on typed values (i32, i64, f32, f64). Instructions map closely to hardware operations.

**Categories:**

| Category             | Description                                    | Stack Input | Stack Output | Examples                         |
| -------------------- | ---------------------------------------------- | ----------- | ------------ | -------------------------------- |
| **Constants**        | Return a fixed value                           | –           | 1 value      | `i32.const 42`, `f32.const 3.14` |
| **Unary operators**  | Consume 1 operand, produce 1 result            | 1           | 1            | `i32.clz`, `f32.sqrt`            |
| **Binary operators** | Consume 2 operands, produce 1 result           | 2           | 1            | `i32.add`, `f64.mul`             |
| **Tests**            | Operate on 1 operand, produce Boolean (`i32`)  | 1           | 1            | `i32.eqz`                        |
| **Comparisons**      | Operate on 2 operands, produce Boolean (`i32`) | 2           | 1            | `i32.lt_s`, `f32.ge`             |
| **Conversions**      | Convert between types                          | 1           | 1            | `i32.wrap_i64`, `f32.demote_f64` |

Integer instructions may have signed (`_s`) or unsigned (`_u`) variants, e.g. `i32.div_u`.

**Examples by type**:

- Integers

```wat
;; Constants
i32.const 42

;; Unary
i32.clz  ;; leading zeros
i32.ctz  ;; trailing zeros
i32.popcnt

;; Binary
i32.add
i32.sub
i32.mul
i32.div_s
i32.div_u
i32.rem_s
i32.rem_u
i32.and
i32.or
i32.xor
i32.shl
i32.shr_s
i32.shr_u
i32.rotl
i32.rotr

;; Tests
i32.eqz

;; Comparisons
i32.eq
i32.ne
i32.lt_s
i32.lt_u
i32.gt_s
i32.gt_u
i32.le_s
i32.le_u
i32.ge_s
i32.ge_u
```

- Floating-point

```wat
;; Constants
f32.const 3.14
f64.const 2.718

;; Unary
f32.neg
f32.abs
f32.sqrt
f32.ceil
f32.floor
f32.trunc
f32.nearest

;; Binary
f32.add
f32.sub
f32.mul
f32.div
f32.min
f32.max
f32.copysign

;; Comparisons
f32.eq
f32.ne
f32.lt
f32.gt
f32.le
f32.ge
```

- Conversions

```wat
i32.wrap_i64        	;; i64 → i32
i64.extend_i32_s    	;; signed i32 → i64
i64.extend_i32_u    	;; unsigned i32 → i64
f32.demote_f64      	;; f64 → f32
f64.promote_f32     	;; f32 → f64
i32.trunc_f32_s     	;; truncate f32 → signed i32
i32.trunc_f32_u     	;; truncate f32 → unsigned i32
f32.reinterpret_i32 	;; bitwise reinterpret i32 as f32
```

