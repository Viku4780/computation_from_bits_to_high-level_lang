
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