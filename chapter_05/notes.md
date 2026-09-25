
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


### ADD

Opcode:

0001

Meaning:

destination = source1 + source2

The source operands can be:

register + register

or:

register + immediate

The destination is always a register.

The book gives the ADD format and explains that bit [5] determines where the second source is a register or an immediate value


### ADD — register form
Example:
```
ADD R2, R0, R1
```

means:
```
R2 = R0 + R1
```

Suppose:
```
R0 = 7
R1 = 3
```

then:
```
R2 = 10
```

The instruction encoding conceptually looks like:
```
0001
 ↓
ADD opcode

000
 ↓
destination R2

000
 ↓
source R0

0
 ↓
register mode

00
 ↓
unused/reserved in this format

001
 ↓
source R1
```

### ADD — immediate form
Example:
```
ADD R2, R0, #5
```

means:
```
R2 = R0 + 5
```

Now bit [5] is:
```
1
```

which tells the hardware:

    The second operand isn't another register. It's the 5-bit immediate field.


### AND
Opcode:
```
0101
```

The instruction structure is almost identical to ADD.

The difference is:
```
ADD → arithmetic addition

AND → bit-by-bit logical AND
```

Example:
```
R0 = 1100 0011
R1 = 1010 0101
```

Then:
```
1100 0011
AND
1010 0101
-----------
1000 0001
```

### NOT
Opcode:
```
1001
```

Unlike ADD and AND, NOT only has one actual source operand.

Example:
```
NOT R3, R5
```

means:
```
R3 = NOT R5
```

If:
```
R5 = 0101 0000 1111 0000
```

then:
```
R3 = 1010 1111 0000 1111
```

### Notice something important about NOT
You might ask:

    “Why are bits [5:0] all 1?”

Because the LC-3 instruction format reserves those bits in NOT, and the ISA specifies that they must be:
```
111111
```

This is a common architectural technique:

    Some bits exist because the instruction encoding has to maintain a fixed format, even when the instruction doesn't need all of them for meaningful operands.


## LEA — Load Effective Address

Opcode:
```
1110
```

LEA does not read memory.

It calculates an address and places that address into a register.

The book even notes that CEA (“Compute Effective Address”) would arguably be a clearer name, but LEA is the industry-established name

### LEA example
Suppose:
```
PC after increment = x4019
offset = -3
```

Then:
```
x4019 + (-3)
=
x4016
```

So:
```
LEA R5, #-3
```

produces:
```
R5 = x4016
```

No memory is accessed.


### Why is LEA useful?
Suppose you want:
```
R3 = address of some data
```

rather than:
```
R3 = contents of that data
```

LEA gives you the address.

That's extremely important for:
```
arrays
strings
tables
data structures
pointers
```

### LEA vs LD
Suppose:
```
M[x5000] = 25
```

and suppose an instruction refers to x5000.

#### LEA
gives you:
```
R1 = x5000
```

#### LD
gives you:
```
R1 = 25
```

So:
```
LEA → give me the ADDRESS

LD  → give me the VALUE at that ADDRESS
```

## Data movement
The LC-3 has six data movement instructions:
```
LD
ST
LDI
STI
LDR
STR
```

Their job is moving data:
```
memory <-> registers
```

### Load vs Store

#### Load
```
memory → register
```

#### Store
```
register → memory
```

Think:
```
LOAD:
"bring something to me"

STORE:
"put something away"
```

### Basic LD

Suppose:
```
M[x4000] = 25
```

and we want:
```
R2 = 25
```

We use:
```
LD R2, ...
```

The machine:
```
calculate memory address
        ↓
      MAR
        ↓
     memory
        ↓
      MDR
        ↓
       R2
```

## PC-relative addressing
The LC-3's LD and ST use:
PC-relative addressing

The formula is:
```
effective address
= 
incremented PC
+
sign-extended PCoffset9
```

Notice:

    incremented PC

not the original PC.

The book explicitly explains that the incremented PC is the value after the FETCH phase has incremented PC.

### Why use PC-relative addressing?

Imagine your program and its data are fairly close together.

Instead of writing the full 16-bit memory address inside every instruction, you can say:
```
"Go 20 locations forward from the current instruction."
```

That's what the offset does.

This makes the instruction encoding more efficient.

### Range of LD/ST
The offset has:
```
9 bits
```

and is a 2's-complement number.

So:
```
-256 ... +255
```

But because the exact interpretation is relative to the incremented PC, the book describes the reachable locations relative to the instruction as approximately:
```
+256 or -255
```

The important idea is:

    PC-relative addressing is local.

If your data is very far away, use another addressing mode.


### Example
Suppose the instruction is at:
```
x4018
```

During FETCH:
```
PC becomes x4019
```

Suppose:
```
PCoffset9 = -81
```

Then:
```
effective address
=
x4019 - 81
```

and that becomes the memory location to access.

The exact arithmetic can be done in binary/hexadecimal, but the mental model is simply:
```
current instruction neighborhood
            +
small signed distance
            ↓
target memory location
```

### LD
Conceptually:
```
LD DR, PCoffset9
```

means:
```
address = PC + offset
DR = M[address]
```

And the loaded value updates:
```
N
Z
P
```

The book explicitly notes that LD sets the condition codes according to whether the loaded value is negative, zero, or positive.

### ST
ST is the opposite direction.

Conceptually:
```
ST SR, PCoffset9
```

means:
```
address = PC + offset
M[address] = SR
```

Notice the register field is interpreted as:
```
SR
```

because the register is the source of the value.

For LD, it is:
```
DR
```

because the register is the destination.

### Why do LD and ST use the same offset mechanism?

Because both need to calculate a memory address.

The difference is what happens once the address is known:
```
LD:
memory → register

ST:
register → memory
```

That's why their instruction formats are similar.


## Indirect Addressing
Suppose:
```
instruction
     ↓
find address A
     ↓
memory[A] contains another address B
     ↓
go to memory[B]
```

That is:
Indirect addressing


### LDI
LDI means:

    load indirectly.

The book describes it this way:

1. Form an address using PC-relative addressing.
2. Read memory at that address.
3. Treat the value read as another address.
4. Read memory at that second address.
5.Put the final value into the destination register.

So conceptually:
```
PC + offset
     ↓
    M[A]
     ↓
    B
     ↓
   M[B]
     ↓
    R
```

### Why is LDI useful?
Because your final data can be anywhere in memory.

PC-relative LD can only reach a limited nearby region

with LDI, the instruction only needs to reach a nearby pointer.

That pointer can point anywhere

This is conceptually related to:
```
pointer
```

in C


### STI
STI is the store version of indirect addressing.

Conceptually:
```
PC + offset
     ↓
memory
     ↓
ADDRESS
     ↓
store register value there
```

So:
```
STI SR, ...
```

means:

    Find a nearby memory location containing an address, then store SR into that address.

The book groups STI with LDI as the indirect addressing pair.

## Base + offset
Suppose an address is already stored in a register:
```
R2 = x2345
```

and we want nearby data:
```
x2362
```

The difference is:
```
x2362 - x2345 = x1D
```

So we can use:
```
LDR R1, R2, #x1D
```

This is:
Base + Offset addressing


### Base + Offset formula
```
effective address
=
contents of BaseR
+
sign-extended offset6
```

The base register is specified by
```
bits [8:6]
```

and the offset uses:
```
bits [5:0]
```

### LDR
LDR means:
```
Load Register
```

Conceptually:
```
address = R[BaseR] + SEXT(offset6)
DR = M[address]
```

Example:
```
R2 = x2345
offset = x1D
```

Then:
```
address = x2345 + x001D
        = x2362
```

Then:
```
R1 = M[x2362]
```

### Why is Base + Offset so important?
Because it is excellent for structures like:
```
arrays
records
stack frames
objects
```

Imagine:
```
R2 = address of array
```

Then:
```
R2 + offset
```

lets you access things relative to that base.

This is a foundational idea for understanding pointer-based memory access later.


### STR
STR is simply the store version
```
STR SR, BaseR, offset6
```

means:
```
address = R[BaseR] + SEXT(offset6)
M[address] = R[SR]
```

So:
```
LDR -> memory -> register
STR -> register -> memory
```

```
REGISTER
    ↓
operand is in register

IMMEDIATE
    ↓
operand is literally inside instruction

PC-RELATIVE
    ↓
PC + offset → memory operand

INDIRECT
    ↓
PC + offset → memory gives address → memory operand

BASE+OFFSET
    ↓
Base register + offset → memory operand
```


## Condition Codes
The LC-3 has three one-bit registers:
```
N
Z
P
```

meaning:
```
N = negative
Z = zero
P = positive
```

The book says these are updated whenever one of the eight general-purpose registers is written as the result of an operate instruction or a load instruction.

### Why do we need condition codes?
Suppose:
```
R2 = 5
```

and you execute:
```
ADD R2, R2, #-5
```

Now:
```
R2 = 0
```

The processor needs some way for the next instruction to ask:

    “Was the result zero?”

That's what:
```
Z = 1
```

tells it.

### Condition codes are basically "what happened?"
Think:
```
previous instruction
       ↓
produced result
       ↓
N/Z/P remember its sign category
```

Then:
```
BR
```

can ask:

    “Should I branch based on that previous result?”

This is the bridge between arithmetic and control flow.


## Branching

### BR
Opcode:
```
0000
```

format:
```
0000
n z p
PCoffset9
```

The three bits:
```
n z p
```

tell the branch instruction which condition codes to examine

### BRz
Suppose:
```
BRz label
```

the z means:

    Branch if Z is set

SO conceptaully:
```
if z == 1:
    PC = target
else:
    continue normally
```

### BRn
```
BRn LABEL
```

means:
```
if N == 1:
    branch
```

### BRp
```
BRp LABEL
```

means:
```
if P == 1:
    branch
```

You can even combine them

### BRnzp is effectively unconditional
What if:
```
n = 1
z = 1
p = 1
```

?

Then the branch checks:
```
N OR Z OR P
```

At least one must be true.

Therefore:
```
BRnzp
```

always branches.

So:
BRnzp = unconditional branch

### What if all three bits are zero?
Then:
```
BR
```

has:
```
n = 0
z = 0
p = 0
```

It checks nothing.

Therefore no branch is taken.

This effectively behaves like a no-op in the sense of leaving the normal sequential flow unchanged, although the instruction still goes through its processing.

### How does a branch calculate its target?
Exactly like PC-relative addressing:
```
target
=
incremented PC
+
SEXT(PCoffset9)
```

The book describes the branch address generation this way.

This is why:
```
BR
```

can only jump a limited distance.


### The range problem
A 9-bit 2's-complement offset gives:
```
-256 ... +255
```

So the branch cannot jump arbitrarily far.

If you need to go thousands of locations away:
```
BR
```

may not be enough.

That's why we have:
JMP