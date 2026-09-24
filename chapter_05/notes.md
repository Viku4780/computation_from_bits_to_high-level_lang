
                 LC-3
                  │
        ┌─────────┴─────────┐
        │                   │
      DATA                INSTRUCTIONS
        │                   │
     registers            opcode
     memory               operands
                           │
                   ┌───────┴────────┐
                   │                │
              where is data?    what operation?
                   │                │
              addressing          opcode
                mode


```
5.1  ISA overview
5.2  Operate instructions
5.3  Data movement instructions
5.4  Control instructions
5.5  Complete example
5.6  Datapath revisited
```

## LC-3

16-bit computer
```
word size       = 16 bits
memory address  = 16 bits
memory location = 16 bits
GPRs            = R0 ... R7
number of GPRs  = 8
```

### The LC-3 Registers
There are eight general-purpose registers:
```
R0
R1
R2
R3
R4
R5
R6
R7
```

why only eight?

Because this is a teaching machine. Eight is enough to demonstrate the important ideas while keeping the instruction encoding manageable.


### What register really is?
A register is simply a group of flip-flops

one flip-flop:
```
1 bit
```

Sixteen flip-flops:
```
16-bit register
```

### What makes an instruction an instruction?
Every LC-3 instruction is:
```
16 bits
```

The basic structure is:
```
┌──────────────┬─────────────────────┐
│    opcode    │   rest of fields    │
│    4 bits    │      12 bits        │
└──────────────┴─────────────────────┘
```

The top four bits:
```
[15:12]
```

tell the processor:

    What kind of instruction is this?

The remaining bits tell it things such as:
```
which register?
which other register?
immediate value?
memory offset?
branch condition?
```

### What is an opcode?

Think of an opcode as a verb.

For example:
```
ADD
```

means:

    add

```
AND
```

means:

    bitwise AND

```
LD
```

means:

    load from memory

```
BR
```

means:

    maybe change where execution goes

So:
```
opcode = WHAT SHOULD THE COMPUTER DO?
```

### What are operands?

Operands are the things the operation works on.

For example:
```
ADD R2, R0, R1
```

means conceptually:
```
R2 ← R0 + R1
```

So:
```
R0 = source
R1 = source
R2 = destination
```

The book describes an ADD instruction as having two source operands and one destination operand.


### The LC-3 instruction families
The LC-3 has 15 defined opcodes. The book divides them into three broad categories:

#### Operate
Compute something.
```
ADD
AND
NOT
```

#### Data movement
Move data between memory and registers.
```
LD
ST
LDI
STI
LDR
STR
```

#### Control
Change the normal sequence of execution.
```
BR
JMP
JSR
JSRR
TRAP
RTI
```

There is also one reserved opcode:
```
1101
```

The book says the LC-3 has 15 instructions even though 4 opcode bits can encode 16 possibilities, leaving 1101 reserved.


### an instruction has both an operation and an interpretation
Suppose a register contains:
```
0011000100110000
```

Those same 16 bits could have different meanings under different representations.

But suppose you execute:
```
ADD R4, R3, #10
```

The ADD instruction tells the hardware:

    Treat the operands as 16-bit 2's-complement integers.

So the hardware does not say:

“What did the programmer intend these bits to mean?”

It follows the meaning associated with the opcode/data type defined by the ISA.

The book gives precisely this example and explains why a bit pattern intended by a programmer as an ASCII value would nevertheless be processed as a 2's-complement integer by ADD


## Addresing modes
Suppose an instruction needs an operand.

Where is that operand?

it could be:
```
inside a register
```

or:
```
inside the instruction itself
```

or:
```
in memory
```

the method used to determine where the operand is located is called the:
Addressing mode

The book identifies five LC-3 addressing modes:
```
1. immediate
2. register
3. PC-relative
4. indirect
5. Base + offset
```

### Register addressing
Example:
```
ADD R1, R2, R3
```

The operands are:
```
R2
R3
```

so the processor doesn't need to search memory.

it simply accesses the registers.

```
R2 ───┐
      │
      ▼
     ALU
      ▲
      │
R3 ───┘
```

That's register addressing.


### Immediate addressing
Now consider:
```
ADD R1, R2, #5
```

The 5 isn't stored in another register

it's literally encoded inside the instruction.

that's why it's called:
```
immediate
```

or :
```
literal
```

### Why do immediate operands exists?
Because sometimes you just need a small constant.

Suppose:
```
R2 = 10
```

and you want:
```
R2 = R2 + 1
```

It would be inconvenient to first put 1 somewhere else.

So:
```
ADD R2, R2, #1
```

lets the instruction carry the value.

#### But immediate values are limited

The instruction only has a limited number of bits available for the immediate.

For ADD and AND:
```
bits [4:0]
```

hold a 5-bit immediate.

Five-bit 2's-complement range:
```
-16 through +15
```

because:
```
-2^4 = -16
2^4 - 1 = 15
```

The book points out that not every integer can therefore be used as an immediate operand.
