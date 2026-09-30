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


### Numeric notation
LC-3 assembly needs a way to tell the assembler what base a number is written in.

The book uses:
```
#   decimal
x   hexadecimal
b   binary
```

Examples:
```
#10
```

means decimal 10.

```
x10
```
means hexadecimal 0x10, which is decimal 16.

```
b1010
```

means binary 1010, which is decimal 10.

This matters because:
```
1000
```

by itself is ambiguous.

The book illustrates:
```
x1000 = decimal 4096
b1000 = decimal 8
#1000 = decimal 1000
```


## Now the first complete assembly program
```
.ORIG x3050

LD R1,SIX
LD R2,NUMBER
AND R3,R3,#0

AGAIN ADD R3,R3,R2
      ADD R1,R1,#-1
      BRp AGAIN

HALT

NUMBER .BLKW 1
SIX    .FILL x0006

.END
```

its goal is:
```
NUMBER x 6
```

by repeated addtion.

This is something you already encountered in chapter 4/5

### .ORIG x3050
This is not a processor instruction.

It is an instruction to the assembler.

It means:

    Start placing the resulting program at memory address x3050.

So the assembler's first location becomes:
```
LC = x3050
```

Here LC means:

    Location Counter

We'll come back to it because it is extremely important.

The .ORIG directive tells the assembler where the program starts.

### LD R1,SIX
This one is a real LC-3 instruction.

At runtime it means:
```
R1 ← M[address of SIX]
```

SIX is a label.

The assembler will eventually discover something like:
```
SIX → x3058
```
and convert the symbolic instruction into the corresponding machine instruction.

So the processor doesn't actually see:
```
LD R1,SIX
```
It eventually sees 16 bits.


### LD R2,NUMBER 
this is also same as previous one

### AND R3,R3,#0
It produces:
```
R3 = 0
```

The comment tells us:
```
R3 will contain the product.
```

So R3 is the accumulator.

### AGAIN
This is not an instruction.

It is a label.

It marks the address where the next instruction lives:
```
AGAIN ADD R3,R3,R2
```

Therefore:
```
AGAIN → address of ADD instruction
```

Later:
```
BRp AGAIN
```

means:

    if positive, branch back to that memory location.


### ADD R3,R3,R2
Now the product starts accumulating.

Suppose:
```
NUMBER = 123
R2 = 123
R3 = 0
```

First iteration:
```
R3 = 0 + 123
   = 123
```

Second:
```
R3 = 246
```

Third:
```
R3 = 369
```
and so on.

### ADD R1,R1,#-1
R1 started with:
```
6
```

So:
```
6 → 5 → 4 → 3 → 2 → 1 → 0
```
The branch will use the resulting condition code.

### BRp AGAIN
This means:
```
if P = 1
    go to AGAIN
```

Since the previous ADD modifies condition codes:
```
R1 ← R1 - 1
```
the branch tests whether the new R1 is positive.

So:
```
R1 = 5 → P → branch
R1 = 4 → P → branch
...
R1 = 1 → P → branch
R1 = 0 → Z → no branch
```
This repeats the addition six times.


### HALT
This is an actual LC-3 instruction.

The processor eventually executes it and stops the program.

Notice the difference between:
```
HALT
```

and:
```
.END
```

This is extremely important.

HALT

Runtime instruction.

The processor executes it.

.END

Assembler directive.

The processor never executes it.


### .BLKW 1
.BLKW means:

    Block of Words

It tells the assembler to reserve a number of sequential memory locations.

For:
```
NUMBER .BLKW 1
```

the assembler reserves one word for NUMBER.

So conceptually:
```
NUMBER
   ↓
one memory location
```

The actual value doesn't have to be known when assembling.

Another piece of the program could later put a value there.

The book specifically identifies .BLKW as useful when the value isn't yet known.


### .FILL x0006
.FILL tells the assembler:

    Put this value directly into the next memory location.

Therefore:
```
SIX .FILL x0006
```

produces:
```
SIX → memory location
M[SIX] = x0006
```

This is important:
```
.FILL
```

doesn't create an instruction.

It creates data in memory.

The book defines .FILL exactly this way


### .STRINGZ
This is another extremely useful pseudo-op.

Example:
```
MESSAGE .STRINGZ "Hello"
```

The assembler stores the ASCII codes of:
```
H
e
l
l
o
```

in consecutive memory locations and then places:
```
0
```

after them.

Conceptually:
```
'H'
'e'
'l'
'l'
'o'
'\0'
```

The book calls the final zero a convenient sentinel.

For the example:
```
HELLO .STRINGZ "Hello, World!"
```

the assembler creates one word per character, followed by x0000.


### .END
.END says:

    The source program is finished.

The assembler stops processing.

But:
```
.END ≠ HALT
```

Think:
```
.END
↓
message to assembler

HALT
↓
instruction executed by processor
```

After assembly, .END is gone.

The book explicitly stresses that .END does not stop execution and does not even exist at runtime.


## instructions vs pseudo-ops
This distinction should become automatic.

#### Real instruction
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
...
```

These correspond to LC-3 ISA operations.

They become machine instructions.

#### Pseudo-op
```
.ORIG
.FILL
.BLKW
.STRINGZ
.END
```

These are messages to the assembler.

They do not correspond to CPU operations.

The book calls them assembler directives or pseudo-ops.

### Five pseudo-ops you need to know
```
| Pseudo-op  | Meaning                                      |
| ---------- | -------------------------------------------- |
| `.ORIG`    | where to place the program                   |
| `.FILL`    | place one specified word in memory           |
| `.BLKW`    | reserve a block of words                     |
| `.STRINGZ` | place ASCII characters plus terminating zero |
| `.END`     | end of source program                        |
```

### Now comes the really important part: the assembler
We now have:
```
program.asm
```

containing:
```
ADD R3,R3,R2
BRp AGAIN
NUMBER .BLKW 1
```

But the LC-3 hardware doesn't understand that textual representation.

The assembler's job is:
```
assembly source
       ↓
machine language
```

The book explicitly defines assembly as the translation process and the assembler as the program that performs it.

### Why can't the assembler translate everything immediately?
Suppose we have:
```
LD R3,PTR
```

The assembler needs to know:

    What memory address does PTR represent?

But perhaps PTR appears much later:
```
PTR .FILL x4000
```

At the moment the assembler is processing:
```
LD R3,PTR
```

it might not yet know the address of PTR.

This is called a forward reference.

The book uses exactly this problem to explain why LC-3 assembly is performed in two passes.

### Why two passes solve the problem
Instead of trying to do everything at once:
```
Pass 1
→ find where all labels are

Pass 2
→ translate instructions using those addresses
```

So:
```
SOURCE
  ↓
PASS 1
  ↓
SYMBOL TABLE
  ↓
PASS 2
  ↓
MACHINE CODE
```

This is the heart of the assembly process.

### Pass 1 — create the symbol table
The symbol table is basically:
```
symbol name → memory address
```

For example:
```
TEST      → x3004
GETCHAR   → x300B
OUTPUT    → x300E
ASCII     → x3012
PTR       → x3013
```

The book gives exactly this symbol table for its character-count example.

### What is the Location Counter?
During pass 1, the assembler needs to know:

    Which memory address are we currently assigning?

It uses the location counter, usually called:
```
LC
```

This is one of those concepts that becomes much easier with a concrete trace.

Suppose:
```
.ORIG x3000
```

Then:
```
LC = x3000
```

Now imagine:
```
AND R2,R2,#0
LD R3,PTR
TRAP x23
LDR R1,R3,#0
```

The assembler conceptually does:
```
LC = x3000
```

Assign first instruction:
```
x3000 → AND
```

then:
```
LC = x3001
```

next:
```
x3001 → LD
```

then:
```
LC = x3002
```

and so on.

The book describes the first pass using precisely this location-counter mechanism.

### Let's actually walk through Pass 1
Use the character-count program from the book:
```
.ORIG x3000

AND R2,R2,#0
LD R3,PTR
TRAP x23
LDR R1,R3,#0

TEST ADD R4,R1,#-4
BRz OUTPUT

NOT R1,R1
ADD R1,R1,#1
ADD R1,R1,R0
BRnp GETCHAR
ADD R2,R2,#1

GETCHAR ADD R3,R3,#1
LDR R1,R3,#0
BRnzp TEST

OUTPUT LD R0,ASCII
ADD R0,R0,R2
TRAP x21
TRAP x25

ASCII .FILL x0030
PTR .FILL x4000

.END
```

Let's trace only addresses.

Start:
```
LC = x3000
```

Then:
```
AND R2,R2,#0
```

gets:
```
x3000
```

Next:
```
LD R3,PTR
```

gets:
```
x3001
```

Then:
```
TRAP x23
```

gets:
```
x3002
```

Then:
```
LDR R1,R3,#0
```

gets:
```
x3003
```

Then:
```
TEST ADD R4,R1,#-4
```

gets:
```
x3004
```

Therefore:
```
TEST → x3004
```

Continue.

Eventually:
```
GETCHAR → x300B
OUTPUT  → x300E
ASCII   → x3012
PTR     → x3013
```

The book produces exactly these symbol-table entries.

### Notice what Pass 1 did NOT do
Very important.

Pass 1 is mainly interested in:
```
Where is everything?
```

It is not primarily trying to generate the final binary instruction.

For example, when it encounters:
```
LD R3,PTR
```

it doesn't yet need to finish translating the instruction.

It needs the information:
```
Where is PTR?
```

### Pass 2 — actually translate
Now we go through the program again.

This time the assembler already has:
```
PTR = x3013
TEST = x3004
GETCHAR = x300B
...
```

So it can translate instructions referencing labels.

This is the second pass.

### Let's deeply understand LD R3,PTR
Suppose:
```
LD R3,PTR
```

is located at:
```
x3001
```

and:
```
PTR = x3013
```

From Chapter 5 you already know:
```
LD uses PC-relative addressing
```

Therefore:
```
EA = incremented PC + PCoffset9
```

At x3001:
```
PC' = x3002
```

because the current instruction's PC is incremented during fetch.

We want:
```
EA = x3013
```

Therefore:
```
x3013 = x3002 + offset
```

So:
```
offset = x3013 - x3002
       = x0011
```

Thus the assembler puts:
```
0010001100010001
```

into memory at x3001.

The book walks through this exact calculation.


### How the assembler thinks
This is a useful mental simulation.

Suppose the assembler sees:
```
BRp AGAIN
```

It effectively thinks:
```
What operation?
→ BRp

What is target?
→ AGAIN

Where is AGAIN?
→ symbol table says x3053

Where is this instruction?
→ x3055

What is incremented PC?
→ x3056

What offset reaches x3053?
→ x3053 - x3056
→ -3

Can -3 fit in PCoffset9?
→ yes
```

Encode bits.

Then it produces the 16-bit instruction.

So assembly language is largely:

    Human-readable description + assembler performs the bookkeeping.

### Why labels are so useful
Imagine you write:
```
BRp x3053
```

That works.

But now insert three instructions earlier.

The target might move from:
```
x3053
```

to:
```
x3056
```

Now you would have to manually update the branch offset/address.

With:
```
BRp AGAIN
```

you don't care.

The assembler recalculates the relationship.

That's why symbolic addresses make programs much easier to maintain.

### But there is still a range limitation
A label does not magically make every addressing mode capable of reaching every address.

Suppose:
```
LD R3,PTR
```

uses PCoffset9.

That offset field has limited range.

So even if the assembler knows where PTR is, the instruction may still be impossible to encode if PTR is too far away.

The book explicitly notes that if the target lies outside the range of the 9-bit PC-relative offset, assembly produces an error.

This gives you an important mental distinction:
```
Label resolution
≠
unlimited addressing
```

The assembler can calculate the address.

But the instruction format still determines whether the result fits.

### Assembly does not change the ISA
Suppose:
```
ADD R3,R3,R2
```

becomes:
```
0001...
```

The assembler didn't create a new operation.

It simply translated a human-friendly notation into the existing LC-3 ISA.

Think:
```
ADD
 ↓
symbolic representation
 ↓
assembler
 ↓
0001...
 ↓
same LC-3 instruction
```

The hardware didn't change.

### The entire assembly pipeline
You write:
```
program.asm
   │
   │  human-readable assembly
   ↓
Assembler
   │
   ├── Pass 1
   │     ↓
   │   symbol table
   │
   └── Pass 2
         ↓
       encode instructions
         ↓
machine-language program
         ↓
loaded into memory
         ↓
PC points to starting address
         ↓
FETCH
DECODE
EXECUTE
...
```

This connects almost every previous chapter.

### Assembly language is not the same thing as the assembler
A common beginner confusion is:

#### Assembly language
The language you write:
```
ADD R1,R2,R3
```

#### Assembler
The program that translates it:
```
ADD R1,R2,R3
        ↓
000100...
```

Think:
```
English
   ↓
English reader/interpreter

Assembly language
   ↓
Assembler
```

The language is the input.

The assembler is the translator.
