# Assembly

These are some general notes of the "Assembly programming language". I'm putting that in quotes because there are many different variants of assembly, but these notes contain information about x86 and ARM assembly (32 bits mostly).

## What is assembly?

Assembly is basically the lowest level programming language around today. It is a way of directly communicating with the CPU of a computer without having to write 1s and 0s, thanks to the mnemnonics it provides. There are many many different variants of Assembly depending on what CPU architecture and operating system you are on. I will not be diving into Computer Architecture or Operating Systems too much because those are huge beasts on their own. Assembly has been around since the dawn of time, or around 1947 [[1]](https://albert.ias.edu/server/api/core/bitstreams/d47626a1-c739-4445-b0d7-cc3ef692d381/content) and still has potential use today in extremely low latency systems, integrated circuts and drivers lol. 

## Example assembly program

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

To get started, you are going to need an x86 CPU. Don't be fooled by the `intel_syntax noprefix` line, that doesn't mean you need an Intel CPU, you can use an AMD x86 CPU as well, its just intel who developed the x86 instruction set, which we are using in this assembly program.

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

A register is a "quickly accessible location available to a computer processor". The x86 instruction set has a lot of different registers for different purposes, and I'm gonna be real I have no clue what half of them are for, but here is a small summary of them [[2]](https://wiki.osdev.org/CPU_Registers_x86)

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

***W.I.P***