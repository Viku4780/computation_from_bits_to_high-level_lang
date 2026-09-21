# The von Neumann Model

```
                    ┌──────────────────────┐
                    │       MEMORY         │
                    │                      │
                    │ programs + data      │
                    └──────────┬───────────┘
                               │
                               │
                    ┌──────────▼───────────┐
                    │      PROCESSOR       │
                    │                      │
                    │  ┌───────────────┐   │
                    │  │ Control Unit  │   │
                    │  └───────────────┘   │
                    │          │           │
                    │  ┌───────▼───────┐   │
                    │  │      ALU      │   │
                    │  └───────────────┘   │
                    │          │           │
                    │     Registers        │
                    └──────────┬───────────┘
                               │
                 ┌─────────────┴─────────────┐
                 ▼                           ▼
              INPUT                        OUTPUT
             keyboard                       monitor
```

The five components in the book's von Neumann model are:

```
1. Memory
2. Processing unit
3. Input
4. Output
5. Control unit
```

The key idea is that  

    the program itself is stored in memory along with data, and the control unit directs the execution of that program instruction by instruction.


## What problem is von Neumann solving?
Imagine a computer that has no stored program.

Suppose you build hardware specifically for:
```
addition
```

Then you want it to do:
```
sorting
```

You would somehow need to change its hardware.

That's inconvenient.

The von Neumann idea is:

    Keep the machine, but store the instructions that tell the machine what to do in memory.

Then:
```
same hardware
    +
different instructions in memory
    +
different behavior
```

    the program and the data are both stored as sequences of bits in memory, and the program is executed one instruction at a time under control of the control unit.

So memory might conceptually look like:
```
Address     Contents
──────────────────────────
x3000       instruction
x3001       instruction
x3002       instruction
x3003       data
x3004       data
x3005       data
```

Notice:
```
instruction
```

and:
```
data
```

are both just bit patterns in memory.

The meaning depends on how the processor uses them.

The processor's current activity and control logic determine how those bits are interpreted