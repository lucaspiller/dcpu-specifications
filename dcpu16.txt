DCPU-16 Specification
Copyright 1985 Mojang
Version 1.7



=== SUMMARY ====================================================================

* 16 bit words
* 0x10000 words of ram
* 8 registers (A, B, C, X, Y, Z, I, J)
* program counter (PC)
* stack pointer (SP)
* extra/excess (EX)
* interrupt address (IA)

In this document, anything within [brackets] is shorthand for "the value of the
RAM at the location of the value inside the brackets". For example, SP means
stack pointer, but [SP] means the value of the RAM at the location the stack
pointer is pointing at.

Whenever the CPU needs to read a word, it reads [PC], then increases PC by one.
Shorthand for this is [PC++]. In some cases, the CPU will modify a value before
reading it, in this case the shorthand is [++PC].

For stability and to reduce bugs, it's strongly suggested all multi-word
operations use little endian in all DCPU-16 programs, wherever possible.



=== INSTRUCTIONS ===============================================================

Instructions are 1-3 words long and are fully defined by the first word.
In a basic instruction, the lower five bits of the first word of the instruction
are the opcode, and the remaining eleven bits are split into a five bit value b
and a six bit value a.
b is always handled by the processor after a, and is the lower five bits.
In bits (in LSB-0 format), a basic instruction has the format: aaaaaabbbbbooooo

In the tables below, C is the time required in cycles to look up the value, or
perform the opcode, VALUE is the numerical value, NAME is the mnemonic, and
DESCRIPTION is a short text that describes the opcode or value.



--- Values: (5/6 bits) ---------------------------------------------------------
 C | VALUE     | DESCRIPTION
---+-----------+----------------------------------------------------------------
 0 | 0x00-0x07 | register (A, B, C, X, Y, Z, I or J, in that order)
 0 | 0x08-0x0f | [register]
 1 | 0x10-0x17 | [register + next word]
 0 |      0x18 | (PUSH / [--SP]) if in b, or (POP / [SP++]) if in a
 0 |      0x19 | [SP] / PEEK
 1 |      0x1a | [SP + next word] / PICK n
 0 |      0x1b | SP
 0 |      0x1c | PC
 0 |      0x1d | EX
 1 |      0x1e | [next word]
 1 |      0x1f | next word (literal)
 0 | 0x20-0x3f | literal value 0xffff-0x1e (-1..30) (literal) (only for a)
 --+-----------+----------------------------------------------------------------
  
* "next word" means "[PC++]". Increases the word length of the instruction by 1.
* By using 0x18, 0x19, 0x1a as PEEK, POP/PUSH, and PICK there's a reverse stack
  starting at memory location 0xffff. Example: "SET PUSH, 10", "SET X, POP"
* Attempting to write to a literal value fails silently



--- Basic opcodes (5 bits) ----------------------------------------------------
 C | VAL  | NAME     | DESCRIPTION
---+------+----------+---------------------------------------------------------
 - | 0x00 | n/a      | special instruction - see below
 1 | 0x01 | SET b, a | sets b to a
 2 | 0x02 | ADD b, a | sets b to b+a, sets EX to 0x0001 if there's an overflow, 
   |      |          | 0x0 otherwise
 2 | 0x03 | SUB b, a | sets b to b-a, sets EX to 0xffff if there's an underflow,
   |      |          | 0x0 otherwise
 2 | 0x04 | MUL b, a | sets b to b*a, sets EX to ((b*a)>>16)&0xffff (treats b,
   |      |          | a as unsigned)
 2 | 0x05 | MLI b, a | like MUL, but treat b, a as signed
 3 | 0x06 | DIV b, a | sets b to b/a, sets EX to ((b<<16)/a)&0xffff. if a==0,
   |      |          | sets b and EX to 0 instead. (treats b, a as unsigned)
 3 | 0x07 | DVI b, a | like DIV, but treat b, a as signed. Rounds towards 0
 3 | 0x08 | MOD b, a | sets b to b%a. if a==0, sets b to 0 instead.
 3 | 0x09 | MDI b, a | like MOD, but treat b, a as signed. (MDI -7, 16 == -7)
 1 | 0x0a | AND b, a | sets b to b&a
 1 | 0x0b | BOR b, a | sets b to b|a
 1 | 0x0c | XOR b, a | sets b to b^a
 1 | 0x0d | SHR b, a | sets b to b>>>a, sets EX to ((b<<16)>>a)&0xffff 
   |      |          | (logical shift)
 1 | 0x0e | ASR b, a | sets b to b>>a, sets EX to ((b<<16)>>>a)&0xffff 
   |      |          | (arithmetic shift) (treats b as signed)
 1 | 0x0f | SHL b, a | sets b to b<<a, sets EX to ((b<<a)>>16)&0xffff

 2+| 0x10 | IFB b, a | performs next instruction only if (b&a)!=0
 2+| 0x11 | IFC b, a | performs next instruction only if (b&a)==0
 2+| 0x12 | IFE b, a | performs next instruction only if b==a 
 2+| 0x13 | IFN b, a | performs next instruction only if b!=a 
 2+| 0x14 | IFG b, a | performs next instruction only if b>a 
 2+| 0x15 | IFA b, a | performs next instruction only if b>a (signed)
 2+| 0x16 | IFL b, a | performs next instruction only if b<a 
 2+| 0x17 | IFU b, a | performs next instruction only if b<a (signed)
 - | 0x18 | -        |
 - | 0x19 | -        |
 3 | 0x1a | ADX b, a | sets b to b+a+EX, sets EX to 0x0001 if there is an over-
   |      |          | flow, 0x0 otherwise
 3 | 0x1b | SBX b, a | sets b to b-a+EX, sets EX to 0xFFFF if there is an under-
   |      |          | flow, 0x0 otherwise
 - | 0x1c | -        | 
 - | 0x1d | -        |
 2 | 0x1e | STI b, a | sets b to a, then increases I and J by 1
 2 | 0x1f | STD b, a | sets b to a, then decreases I and J by 1
---+------+----------+----------------------------------------------------------

* The branching opcodes take one cycle longer to perform if the test fails
  When they skip an if instruction, they will skip an additional instruction
  at the cost of one extra cycle. This lets you easily chain conditionals.  
* Signed numbers are represented using two's complement.

    
Special opcodes always have their lower five bits unset, have one value and a
five bit opcode. In binary, they have the format: aaaaaaooooo00000
The value (a) is in the same six bit format as defined earlier.

--- Special opcodes: (5 bits) --------------------------------------------------
 C | VAL  | NAME  | DESCRIPTION
---+------+-------+-------------------------------------------------------------
 - | 0x00 | n/a   | reserved for future expansion
 3 | 0x01 | JSR a | pushes the address of the next instruction to the stack,
   |      |       | then sets PC to a
 - | 0x02 | -     |
 - | 0x03 | -     |
 - | 0x04 | -     |
 - | 0x05 | -     |
 - | 0x06 | -     |
 - | 0x07 | -     | 
 4 | 0x08 | INT a | triggers a software interrupt with message a
 1 | 0x09 | IAG a | sets a to IA 
 1 | 0x0a | IAS a | sets IA to a
 3 | 0x0b | RFI a | disables interrupt queueing, pops A from the stack, then 
   |      |       | pops PC from the stack
 2 | 0x0c | IAQ a | if a is nonzero, interrupts will be added to the queue
   |      |       | instead of triggered. if a is zero, interrupts will be
   |      |       | triggered as normal again
 - | 0x0d | -     |
 - | 0x0e | -     |
 - | 0x0f | -     |
 2 | 0x10 | HWN a | sets a to number of connected hardware devices
 4 | 0x11 | HWQ a | sets A, B, C, X, Y registers to information about hardware a
   |      |       | A+(B<<16) is a 32 bit word identifying the hardware id
   |      |       | C is the hardware version
   |      |       | X+(Y<<16) is a 32 bit word identifying the manufacturer
 4+| 0x12 | HWI a | sends an interrupt to hardware a
 - | 0x13 | -     |
 - | 0x14 | -     |
 - | 0x15 | -     |
 - | 0x16 | -     |
 - | 0x17 | -     |
 - | 0x18 | -     |
 - | 0x19 | -     |
 - | 0x1a | -     |
 - | 0x1b | -     |
 - | 0x1c | -     |
 - | 0x1d | -     |
 - | 0x1e | -     |
 - | 0x1f | -     |
---+------+-------+-------------------------------------------------------------



=== INTERRUPTS =================================================================    

The DCPU-16 will perform at most one interrupt between each instruction. If
multiple interrupts are triggered at the same time, they are added to a queue.
If the queue grows longer than 256 interrupts, the DCPU-16 will catch fire. 

When IA is set to something other than 0, interrupts triggered on the DCPU-16
will turn on interrupt queueing, push PC to the stack, followed by pushing A to
the stack, then set the PC to IA, and A to the interrupt message.
 
If IA is set to 0, a triggered interrupt does nothing. Software interrupts still
take up four clock cycles, but immediately return, incoming hardware interrupts
are ignored. Note that a queued interrupt is considered triggered when it leaves
the queue, not when it enters it.

Interrupt handlers should end with RFI, which will disable interrupt queueing
and pop A and PC from the stack as a single atomic instruction.
IAQ is normally not needed within an interrupt handler, but is useful for time
critical code.




=== HARDWARE ===================================================================    

The DCPU-16 supports up to 65535 connected hardware devices. These devices can
be anything from additional storage, sensors, monitors or speakers.
How to control the hardware is specified per hardware device, but the DCPU-16
supports a standard enumeration method for detecting connected hardware via
the HWN, HWQ and HWI instructions.

Interrupts sent to hardware can't contain messages, can take additional cycles,
and can read or modify any registers or memory adresses on the DCPU-16. This
behavior changes per hardware device and is described in the hardware's
documentation.

Hardware must NOT start modifying registers or ram on the DCPU-16 before at
least one HWI call has been made to the hardware.

The DPCU-16 does not support hot swapping hardware. The behavior of connecting
or disconnecting hardware while the DCPU-16 is running is undefined.