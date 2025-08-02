# Assembly

These are some general notes of the "Assembly programming language". I'm putting that in quotes because there are many different variants of assembly, but these notes contain information about x86 and ARM assembly (32 bits mostly).

## What is assembly?

Assembly is basically the lowest level programming language around today. It is a way of directly communicating with the CPU of a computer without having to write 1s and 0s, thanks to the mnemnonics it provides. There are many many different variants of Assembly depending on what CPU architecture and operating system you are on. I will not be diving into Computer Architecture or Operating Systems too much because those are huge beasts on their own. Assembly has been around since the dawn of time, or around 1947 [[1]](https://albert.ias.edu/server/api/core/bitstreams/d47626a1-c739-4445-b0d7-cc3ef692d381/content) and still has potential use today in extremely low latency systems, integrated circuts and hardware drivers. Any compiled programming language, like C or C++ (and unlike Java or Kotlin, which are compiled to Java Bytecode) are first compiled into Assembly programs, which are then used to create the executable.

## Hello Assembly!

Here is what a Hello World program could look like in assembly

```assembly
.intel_syntax noprefix

.data
text: .asciz "Hello World!\n"

.text
.global main
main:
    push ebp
    mov ebp, esp

    push offset text
    call printf
    add esp, 4
    mov eax, 0

    mov esp, ebp
    pop ebp
    ret
```

The reason I said this is what a Hello World program *could* look like, is because there are countless different ways to do this. Also a lot is going on in this code but I will attempt to explain all of this.

**So first off, how do I run this?**

To get started, you are going to need an x86 CPU. Don't be fooled by the `intel_syntax noprefix` line, that doesn't mean you need an Intel CPU, you can use an AMD x86 compatible CPU as well, its just intel who developed the x86 instruction set, which we are using in this assembly program. 

>**Fun fact:** eventually AMD was the first to develop a 64-bit x86 instruction set architecture with their Opteron family of processors in 2003, and this is why sometimes x86_64 is also referred to as AMD64.

Next step is to install Linux on your machine, as you can see we are using `printf` to print to the terminal, plus its very easy to assemble this file using gcc, since the gcc toolchain contains an assembler.

Then you want to create a file for your code, e.g. `helloworld.S`. The `.S` file extention is important, this tells the assembler that this is a hand written assembly source code file, and runs preprocessing on the file (e.g. removing comments). I recommend using nano for editing files, as it is rather easy to use within a terminal.

Once you've written up the file, you can assemble and execute the file
```bash
gcc -m32 -static -g -o helloworld helloworld.S
./helloworld
```

What is this command:
- `gcc`: GNU C Compiler, used for compiling C (obviously), but has an assembler we can hijack and use for our own purposes (printing hello world)
- `-m32`: Ensures that a 32 bit binary is created
- `-static`: Ensures we create a standalone binary that doesn't need anything to be loaded from an external library
- `-g`: Makes it easier to debug a program using `gdb`, which can be very useful, especially when working with assembly
- `-o helloworld`: Specifies the name of the `output` binary we can run
- `helloworld.S`: Our assembly source code file

**So, whats going on in the code?**

I'm going to try to explain the code in a way that any novice C developer could understand, without explaining the underlying details too much, which we will get into later.

`intel_syntax noprefix`

This line tells the assembler that we are using the "Intel" syntax. The default syntax for gcc is the "AT&T" syntax, which is horrible and looks like this:

```assembly
    push %ebp
    mov %esp, %ebp

    push $text
    call printf
    add $4, %esp
    mov $0, %eax
```

So we use the Intel syntax to make our lives just that bit easier (not much easier, we are still programming in assembly).

```assembly
.data
text: .asciz "Hello World!\n"
```
This `.data` section is where we put our data. These are constants, so their value cannot be changes during the program. This is basically just a human readable way to reference data we put into memory.

```assembly
.text
.global main
main:
```
The `.text` section is where we actually write our code. The `.global main` section ensures that our program ("function") is available outside of the current scope. GCC requires that any assembly source code has at least one of these, as this will be our main function (like `int main(int argc, char** argv)`). ``main:`` is where the starting point of our actual program is.

```assembly
    push ebp
    mov ebp, esp
```

This is what is known as the "function prologue". This is an essential part of writing an assembly program, but we will skip this for now.

```assembly
    push offset text
    call printf
```
This is the part where we print the text, we are essentially doing this: `printf(text);`, but to pass printf the argument of text, we need to push `offset text` (memory address of the starting char of `text`) onto the stack, and then `call` printf.

```assembly
    add esp, 4
    mov eax, 0
```
Here we are doing some data manipulation in the *registers* (more on those soon), but why?
 - `add esp, 4`: `esp` is the "stack pointer", more on that later, but the basics are if we do something to the stack (`push`), we must ensure it is set back to how it was before exiting the program, so set the stack pointer to the value it was at when the program started. We can do this in 2 ways, either `pop`-ing as many times as we pushed, or adding to the value of the stack pointer (more on this too l8r). Here since we pushed once, we `add 4` to `esp`, since we pushed a memory address (32 bits, 4 bytes), we add back those 4 bytes to bring it back to "zero".
 - `mov eax, 0`: `eax` is a "general purpose" register, but in this context (cdecl calling convention), this is where we store the return value of the program. this plus `ret` is equivalent to `return 0;`

 ```assembly
    mov esp, ebp
    pop ebp
    ret
```

`mov` and `pop` are part of the "function epilogue", we'll also skip this for now. `ret` is the command to return the program, or exit, with the code stored in `eax`.

**This is stupid, I'm going back to python**

I can see why some people may not see a point in this, but once you imagine that what you are writing is literally directly controlling the CPU, and memory, I think you can gain an affinity for Assembly.

## Registers

**What is a register?**

A register is a "quickly accessible location available to a computer processor". The x86 cpu architecture set has a lot of different registers for different purposes, and I'm gonna be real I have no clue what half of them are for, but here is a general summary of them [[2]](https://wiki.osdev.org/CPU_Registers_x86) (this is taken from wiki.osdev.org, if someone would like this section removed, please contact me)

### General Purpose Registers

| 64-bit | 32-bit | 16-bit |8 high bits | 8 low bits | Description |
|--------|--------|--------|------------|------------|-------------|
| RAX | EAX | AX | AH  | AL  | Accumulator |
| RBX | EBX | BX | BH  | BL  | Base |
| RCX | ECX | CX | CH  | CL  | Counter |
| RDX | EDX | DX | DH  | DL  | Data |
| RSI | ESI | SI | N/A | SIL | Source |
| RDI | EDI | DI | N/A | DIL | Destination |
| RSP | ESP | SP | N/A | SPL | Stack Pointer |
| RBP | EBP | BP | N/A | BPL | Stack Base Pointer  |

### Pointer Registers

|64-bit | 32-bit | 16-bit |	Description|
|-------|--------|--------|------------|
|RIP | EIP | IP | Instruction Pointer |

### Segment Registers

|16-bit |	Description|
|-------|--------------|
|CS |	Code Segment|
|DS |	Data Segment|
|ES |	Extra Segment|
|SS |	Stack Segment|
|FS |	General Purpose F Segment|
|GS |	General Purpose G Segment |

### Eflags Register
| Bit | Label | Description |
|--|-------|------------|
|0 |	CF | Carry flag |
|2 |	PF | Parity flag |
|4 |	AF | Auxiliary flag |
|6 |	ZF | Zero flag |
|7 |	SF | Sign flag |
|8 |	TF | Trap flag |
|9 |	IF | Interrupt enable flag |
|10|	DF | Direction flag |
|11|	OF | Overflow flag |
|12-13 |	IOPL | I/O privilege level |
|14 | NT  |	Nested task flag |
|16 | RF  |	Resume flag |
|17 | VM  |	Virtual 8086 mode flag |
|18 | AC  |	Alignment check |
|19 | VIF | 	Virtual interrupt flag |
|20 | VIP | 	Virtual interrupt pending |
|21 | ID  |	Able to use CPUID instruction |

### Control Registers

**CR0**

|Bit |	Label |	Description |
|----|--------|-------------|
|0 | PE |	Protected Mode Enable|
|1 | MP |	Monitor co-processor|
|2 | EM |	x87 FPU Emulation|
|3 | TS |	Task switched|
|4 | ET |	Extension type|
|5 | NE |	Numeric error|
|16| WP |	Write protect|
|18| AM |	Alignment mask|
|29| NW |	Not-write through|
|30| CD |	Cache disable|
|31| PG |	Paging |

**CR1**

Reserved, the CPU will throw a #UD exception when trying to access it. 

**CR2**
|Bit |	Label |	Description |
|----|--------|-------------|
|0-31  (63) |	PFLA |	Page Fault Linear Address |

**CR3**
|Bit |	Label |	Description |	PAE |	Long Mode |
|----|--------|-------------|-------|-------------|
|3 |	PWT |	Page-level Write-Through |	(Not used) |	(Not used if bit 17 of CR4 is 1)
|4 |	PCD |	Page-level Cache Disable |	(Not used) |	(Not used if bit 17 of CR4 is 1)
|12-31 (63) | 	PDBR |	Page Directory Base Register |	Base of PDPT |	Base of PML4T/PML5T

Bits 0-11 of the physical base address are assumed to be 0. Bits 3 and 4 of CR3 are only used when accessing a PDE in 32-bit paging without PAE. 

**CR4**
|Bit |	Label |	Description|
|----|--------|------------|
|0 |	VME| 	Virtual 8086 Mode Extensions
|1 |	PVI| 	Protected-mode Virtual Interrupts
|2 |	TSD| 	Time Stamp Disable
|3 |	DE |	Debugging Extensions
|4 |	PSE| 	Page Size Extension
|5 |	PAE| 	Physical Address Extension
|6 |	MCE| 	Machine Check Exception
|7 |	PGE| 	Page Global Enabled
|8 |	PCE| 	Performance-Monitoring Counter enable
|9 |	OSFXSR |	Operating system support for FXSAVE and FXRSTOR instructions
|10| 	OSXMMEXCPT |	Operating System Support for Unmasked SIMD Floating-Point Exceptions
|11| 	UMIP |	User-Mode Instruction Prevention (if set, #GP on SGDT, SIDT, SLDT, SMSW, and STR instructions when CPL > 0)
|12| 	LA57 |	57-bit linear addresses (if set, the processor uses 5-level paging otherwise it uses uses 4-level paging)
|13| 	VMXE |	Virtual Machine Extensions Enable
|14| 	SMXE |	Safer Mode Extensions Enable
|16| 	FSGSBASE |	Enables the instructions RDFSBASE, RDGSBASE, WRFSBASE, and WRGSBASE
|17| 	PCIDE |	PCID Enable
|18| 	OSXSAVE |	XSAVE and Processor Extended States Enable
|20| 	SMEP |	Supervisor Mode Execution Protection Enable
|21| 	SMAP |	Supervisor Mode Access Prevention Enable
|22| 	PKE |	Protection Key Enable
|23| 	CET |	Control-flow Enforcement Technology
|24| 	PKS |	Enable Protection Keys for Supervisor-Mode Pages 

**CR5 - CR7**

Reserved, same case as CR1.

**CR8**
Bit |	Label | 	Description
|---|---------|------------------
0-3 |	TPL |	Task Priority Level 

There are also Extended Control Registers, Debug Registers, etc. I might add them later, but they aren't super important to understand the basics of assembly.

### General Purpose Registers, again

As you can see, each General Purpose Register has a 64-bit (let's pretend this doesn't exist), 32-bit, 16-bit, 8 high bit and 8 low bit "version". These arent actually seperate registers, these are parts of these registers, so for example `AX` is actually just the lower 16 bits of `EAX`, and `AL` is the lower 8 bits of `AX`. Take for example this code:

```Assembly
    mov eax, 0x0                /* eax = 0x00000000 */
    mov ax, 0b1111111111111111  /* we move this binary number to ax, basically setting it all to 1, so eax = 0x0000FFFF */
    mov al, 0x0                 /* we move 0 to the lower 8 bits of ax, so eax = 0x0000FF00 */
```
Here, eax is now `0b00000000000000001111111100000000`, by the way `0x0` and `0b0` are just the ways to denote hexadecimal and binary numbers.

There are 8 "general purpose" registers. But in practive there are only 6 that are actually "truly" general purpose. I say this because `EBP` and `ESP` are the "Stack Base Pointer" and "Stack Pointer", `EBP` points to the base of the current stack frame and is mainly used to carefully navigate function parameters and local variables, while `ESP` points to the "top" of the stack, or to the element that was last added to it.

That said you technically *can* use these as general purpose registers, but doing so in a careless way will lead to some bad times, like stack corruption or segmentation faults (fancy words for saying your program doesn't work).

Other general purpose registers are also bound to some instructions, meaning some instructions will only accept the parameter in a particular register, and/or move the output to a particular register, for example the `MUL` instruction, but we will get into that later.

### Eflags Register, again

The `EFLAGS` register is another important register to know, here flags are stored about the current state of the program, which can be useful if we want to implement conditional branching for example. Some important flags are:

- **CF, Carry Flag**: Set if an arithmetic operation generates a carry out (or borrow in subtraction)
- **PF, Parity Flag**: Set if the number of set bits (1s) in the least significant byte of the result is even.
- **AF, Auxiliary Flag**: Set if there's a carry or borrow from bit 3 to bit 4 (lower nubble to higher nibble, yes half a byte is called a nibble), used in Binary Coded Decimal arithmetic
- **ZF, Zero Flag**: Set if the result of the last operation is zero.
- **SF: Sign Flag**: Set if the result if negative (the most significant bit is 1 in signed arithmetic)

These flags usually aren't modified manually, instead the CPU automatically updates them after most arithmetic or logical instructions.

### EIP Register

The `EIP` register, or Instruction Pointer is a special register which cannot be written to, it stores a pointer to the next instruction, so the CPU knows what to do next.

## The Stack

The stack is an important part of any (yes I mean any!) computer program. When an assembly program is started, it gets a stack, which can be accessed through the `ESP` pointer. The stack also has the Base Pointer, `EBP`, but we won't be using that right now.

The stack is a LIFO data structure if using the `PUSH` and `POP` instructions, meaning what we last put on (push) to the stack is the first thing we can "take off" (pop). This is important to think about when calling functions using the CDECL calling convention, as this calling convention takes parameters from the stack **in reverse order**, but we'll get into that later.

Here is an example of using the `PUSH` instruction
```
        addr     content
       0x0010 | 0x11223344
       0x0014 | 0x00000011
       0x0018 | 0x01000100
ESP -> 0x001C | 0x00112233
       0x0020 | 0x01234567
       0x0024 | 0x00001100
       0x0028 | 0x00000000

PUSH 0xFFFFFFFF

        addr     content
       0x0010 | 0x11223344
       0x0014 | 0x00000011
ESP -> 0x0018 | 0xFFFFFFFF <- new value
       0x001C | 0x00112233
       0x0020 | 0x01234567
       0x0024 | 0x00001100
       0x0028 | 0x00000000
```

And the `POP` instruction:

```
        addr     content
       0x0010 | 0x11223344
       0x0014 | 0x00000011
ESP -> 0x0018 | 0xFFFFFFFF
       0x001C | 0x00112233
       0x0020 | 0x01234567
       0x0024 | 0x00001100
       0x0028 | 0x00000000

POP EAX

        addr     content
       0x0010 | 0x11223344
       0x0014 | 0x00000011
       0x0018 | 0xFFFFFFFF <- value stays here! only pointer is modified
ESP -> 0x001C | 0x00112233
       0x0020 | 0x01234567
       0x0024 | 0x00001100
       0x0028 | 0x00000000
```

As you can see the stack grows towards lower memory addresses, and shrinks towards higher memory addresses.

If you are keen eyed (and remember the example program) you'll notice that it isn't necessary to always `POP` as many times as you `PUSH`. You can simply adjust `ESP` manually, for example instead of writing:

```Assembly
    push eax
    push ebx
    push ecx
    call function
    pop ecx
    pop ebx
    pop eax
```

you can simply do this:

```Assembly
    push eax
    push ebx
    push ecx
    call function
    add esp, 12       /* clean up 3 pushes x 4 bytes each */
```

## Data moving instructions

Time for instructions, which are just as important to an assembly program, since without them we cant really do anything.

### `PUSH`

`push <operand>`

Puts the operand onto the stack

*How it works:*
- The value of `ESP` is lowered by 4 if `<operand>` is 32 bits, or 2 if it is 16 bits
- The value of `<operand>` is copied to the memory address that `ESP` now points to (`MOV [ESP], <operand>`)

So this is technically just:
```Assembly
    sub esp, 4
    mov [esp], <operand>
```

### `POP`

`pop <operand>`

Copies the top of the stack to `<operand>`, then """""removes""""" it from the stack

*How it works:*
- The value of the memory address `ESP` points to is copied to `<operand>`
- The value of `ESP` is raised by 4 if `<operand>` is 32 bits, or 2 if it is 16 bits.

So this is techincally just:
```Assembly
    mov <operand>, [esp]
    add esp, 4
```

**Remember:** Values on the stack are never actually removed when popped, they are only copied to `<operand>` and then the pointer is moved. The data stays there.

***W.I.P***