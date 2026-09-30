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