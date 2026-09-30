    How an assembler takes your human-readable assembly program and turns it into the machine-language instructions and the processor actually executes


## Machine language vs assembly language
Suppose you want this operation:
```
R3 <- R3 + R2
```

In LC-3 machine language, that instruction is a sequence of 16 bits:
```
0001 011 011 0 00 010
```

You already learned how those fields work in Chapter 5.

But imagine wrting a 500-line program like that.

you would have to remember:
```
0001 = ADD
0101 = AND
1001 = NOT
0010 = LD
```

and constantly count bits.

assembly language says:
```
ADD R3, R3, R2
```

That's much easier for humans.

the computer, however, still executes the 16-bit machine instruction.

so:
```
ADD R3,R3,R2
```
is not directly understood by the LC-3 hardware

instead:
```
Assembly program
       ↓
    assembler
       ↓
Machine-language program
       ↓
      LC-3
       ↓
   executes bits
```

### Why assembly language exits

### Problem 1: remembering opcodes
instead of:
```
0001
```

you write:
```
ADD
```

instead of:
```
1001
```

you write:
```
NOT
```
these readable names are called mnemonics.


### Problem 2: remembering memory address
imagine:
```
LD R2, x3057
```
this tells you something, but you still have to remember what x3057 represents

Suppose x3057 contains the number used as input.

you can instead write:
```
LD R2, NUMBER
```

Now the code tells you what that memory location means.

The name NUMBER is a label, or symbolic address.

The assembler later figures out that:
```
NUMBER -> x3057
```


### Assembly language does not hide the machine
this is an important difference between assembly and C.

Consider:
```
x = x + 1;
```

A C programmer does not directly specify which machine instruction performs that operation.

the compiler decides.

But with LC-3 assembly:
```
ADD R1, R1, #1
```

you are explicitly telling the machine:

    Add 1 to R1

you still have detailed control over the underlying ISA.


### Assembly language is ISA-dependent
High-level languages are generally:
```
ISA independent
```

For example, you can write C for:
```
x86
ARM
RISC-V
...
```

But assembly languages are tied closely to a particular ISA.

For example:
```
LC-3 assembly
```

is specifically describing LC-3 instructions.

So:
```
C
↓
can target many different CPUs

LC-3 assembly
↓
describes LC-3
```

That is why assembly is called a low-level language.


### The four parts of an assembly instruction
This is one of the first things you should become comfortable reading.

The book gives the form:
```
Label   Opcode   Operands   ; Comment
```

There are four conceptual pieces:
```
LABEL     OPCODE       OPERANDS       COMMENT
  │          │             │              │
name      operation       data         explanation
```

Only opcode and operands are mandatory.

Label and comment are optional.


### Opcode
The opcode says:

    What operation should be performed?

Examples:
```
ADD
AND
NOT
LD
ST
BR
LDR
STR
LEA
TRAP
```

For example:
```
ADD R3,R3,R2
```

ADD is the opcode.

The assembler knows:
```
ADD → binary opcode 0001
```

because the LC-3 ISA specifies that mapping.


### Operands
Operands tell the instruction:

    What should the operation act on?

Example:
```
ADD R3,R3,R2
```

has three operands:
```
R3
R3
R2
```

Meaning:
```
destination = R3
source 1    = R3
source 2    = R2
```

Therefore:
```
R3 ← R3 + R2
```

The assembler uses these operands to fill the appropriate bit fields in the 16-bit instruction.


### Label
A label is simply a human-readable name for a memory location.

For example:
```
AGAIN ADD R3,R3,R2
```

Here:
```
AGAIN
```

is a label.

It means:

    The memory location containing this instruction has been given the symbolic name AGAIN.

Later you can write:
```
BRp AGAIN
```

instead of manually calculating the numerical address.

The assembler performs that calculation for you.


### Comments
Anything after ; is a comment.

Example:
```
ADD R3,R3,#1 ; increment counter
```

The assembler processes:
```
ADD R3,R3,#1
```

and ignores:
```
; increment counter
```

Comments are for humans, not the processor.

The book makes a useful distinction here: good comments should explain something that isn't obvious rather than merely repeating the instruction.

For example:
```
ADD R1,R1,#-1 ; Decrement R1
```

isn't particularly useful.

But:
```
ADD R1,R1,#-1 ; R1 tracks the number of iterations remaining
```

provides additional information.


### Labels are not variables
This distinction will prevent a lot of confusion.

Suppose:
```
NUMBER .BLKW 1
```

You might be tempted to think:
```
NUMBER is a variable.
```

Conceptually, that's not quite what the label itself is.

NUMBER is a name attached to a memory address.

For example:
```
NUMBER → x3057
```

Then:
```
M[x3057]
```

is the actual memory location.

So think:
```
NUMBER
   ↓
memory address x3057
   ↓
contents stored there
```

A label gives a name to the location, not directly to the data inside it.

### When do we need a label?

#### Reason 1: branch target

Example:
```
AGAIN ADD R3,R3,R2
...
BRp AGAIN
```

AGAIN identifies where execution should go when the branch is taken.

#### Reason 2: data location
Example:
```
NUMBER .BLKW 1
```

and:
```
LD R2,NUMBER
```

Here the label identifies a memory location containing data.

So the same idea works for both:
```
label → memory address
```

### Label naming rules
For the LC-3 assembly language described in the book, a label:

- can contain 1–20 alphanumeric characters
- must begin with a letter
- cannot be something that already has a special meaning in the assembly language.

Examples of valid labels:
```
NOW
Under21
R2D2
R785
C3PO
```

Examples that are not valid labels include things such as:
```
ADD
NOT
R4
```

because they already have special meaning.

The book calls these reserved words.