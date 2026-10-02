Core question:

    How does the processor communicate with outside world, and how does the operating system safely control that communication?


## What does I/O actually mean?
you already know:

Input
```
outside world -> computer
```

examples:
```
keyboard -> CPU
mouse -> computer
sensor -> computer
```

Output
```
computer -> outside world
```

examples:
```
CPU -> monitor
CPU -> printer
CPU -> actuator
```

so:
```
                 COMPUTER
                    |
          +---------+---------+
          |                   |
        INPUT              OUTPUT
          ↑                   ↓
       keyboard            monitor
       sensor              printer
       etc.                etc.
```

At first glance this seems simple.

But there is a major problem:

    The CPU and an I/O device are not necessarily operating at the same speed or at the same time.

That leads us into the first major concept.


### Two concepts: privilage and priority
They are orthogonal, meaning that one is independent of the other.

### Privilage = "Am i allowed to do this?"
Privilage is about permission.

imagine that the computer contains:
```
operating system code
operating system data
user program A
user program B
```

would we want an ordinary user program to do this?
```
erase OS memory
change OS data
stop entire computer
change hardware configuration
```

Obviously not.

Therefore the processor needs a way to say:
```
this program is trusted to do more
this program is restricted
```

The LC-3 has two modes:
```
Supervisor mode
User mode
```

#### Supervised mode
A privilaged program can:
```
execute all instructions
access all memory
```

#### User mode
An unprivilaged program cannot access protected resources

the processor prevents forbidden operations.


```
USER MODE
   |
   | ordinary doors
   ↓
USER RESOURCES
```

```
SUPERVISOR MODE
   |
   | ordinary + restricted doors
   ↓
SYSTEM RESOURCES
```


### Priority = "How urgently must this happen"
Priority is completely different.

Suppose:
```
Program A = normal work
Keyboard = waiting for input
Power failure = happening right now
```

the computer cannot neccessarily treat all three as equally urgent.

so things get priority levels.

The LC-3 has:
```
PL0
PL1
PL2
...
PL7
```
where:
```
PL0 = lowest
PL7 = highest
```

    Priority answer "who needs the processor more urgently?"


## Processor Status Register -- PSR
How does the processor remember privilage and priority?

Through the:
PSR

LC-3 PSR contains:
```
privilage
priority
condition codes
```

the important fields are:
```
bit 15   -> privilage
bits 10:8 -> priority
bits 2:0  -> N Z P condition codes
```

For the LC-3
```
PSR[15] = 0 -> Supervisor
PSR[15] = 1 -> User
```

and:
```
PSR[10:8] = priority level
```

the condition codes are also stored in PSR because they are part of the processor state that must survive certain control transfers.


### Why are the condition codes in the PSR?
Remember your LC-3 branches:
```
BRz
BRn
BRp
BRnz
...
```

They depend on:
```
N
Z
P
```

Now imagine a user program does:
```
ADD R1,R1,#1
BRz SOMETHING
```

Then an interrupt occurs.

The interrupt service routine executes an ADD.

That changes:
```
N/Z/P
```

If the processor did not preserve the original condition codes, the user's BRz might make the wrong decision when the user program resumes.

Therefore:
```
program A condition codes
          ↓
       saved
          ↓
interrupt executes
          ↓
       restored
          ↓
program A continues correctly
```

This is a deep reason why processor state matters.

The chapter explicitly identifies the condition codes, priority, and privilege as state that must be preserved across interrupts.


## Memory organization of the LC-3
Now we need to understand where the OS, programs, and I/O live.

The LC-3 has a 16-bit address:
```
0000 0000 0000 0000
        ...
1111 1111 1111 1111
```

So there are:
```
2^16 = 65,536
```
address values.

That gives:
```
x0000 → xFFFF
```
The LC-3 divides this space into important regions.

### LC-3 address-space map
```
x0000
   │
   │ SYSTEM SPACE
   │ privileged
   │ OS code + OS data
   │
x2FFF
   │
   │ USER SPACE
   │ user programs + user data
   │
xFDFF
   │
   │ I/O PAGE
   │ device registers
   │ processor special registers
   │
xFFFF
```

```
x0000 – x2FFF
    System space
    privileged

x3000 – xFDFF
    User space
    unprivileged

xFE00 – xFFFF
    I/O page / special registers
    privileged
```


### Why do we need two stacks?
Now Chapter 9 gives us two contexts.
```
USER PROGRAM
    ↓
USER STACK
```

and:
```
OPERATING SYSTEM
    ↓
SUPERVISOR STACK
```

Why separate them?

Because when the processor switches into privileged OS handling, the system needs protected stack storage for system-level state.

The LC-3 therefore has:
```
USP = User Stack Pointer
SSP = Supervisor Stack Pointer
```

and:
```
R6
```

is used as the active stack pointer.

When privilege changes, the currently active stack pointer is saved and the other one becomes active.


### Now we meet hardware device registers
A keyboard needs somewhere to put the character.

It also needs some way to say:

    "I have a character ready."

Therefore we need at least two registers:
```
data register
status register
```

The LC-3 keyboard has:
```
KBDR
KBSR
```

where:
```
KBDR = Keyboard Data Register
KBSR = Keyboard Status Register
```
where:
```
KBDR = keyboard data register
KBSR = keyboard status register
```

their addresses are:
```
KBDR = xFE02
KBSR = xFE00
```

### Keyboard Data Register — KBDR
Suppose you press:
```
A
```

The keyboard hardware puts the ASCII value for A into:
```
KBDR[7:0]
```

So conceptually:
```
keyboard
   |
   | "A"
   ↓
KBDR
```

The upper bits aren't needed for the character in this example.


### Keyboard Status Register — KBSR
Now the CPU needs to know:

    "Is there a character waiting?"

That's what the status register tells us.

In the LC-3:
```
KBSR[15]
```

is the ready bit.

Think:
```
KBSR[15] = 0
    ↓
no new character available

KBSR[15] = 1
    ↓
character is ready
```

When a key is pressed:
```
keyboard loads KBDR
        ↓
KBSR[15] = 1
```

When the processor reads the KBDR:
```
character consumed
        ↓
KBSR[15] = 0
```

This is a synchronization mechanism between the slow keyboard and fast processor.

### Why do we need the ready bit?
Imagine the processor is extremely fast:
```
CPU:
read KBDR
read KBDR
read KBDR
read KBDR
...
```

But a human may take:
```
0.5 second
```

to type another key.

Without some status information, the processor doesn't know whether:
```
the character is new
```

or:
```
the same old character is still sitting there
```

So:
```
KBSR[15]
```

acts like a tiny communication signal:
```
Keyboard → "new data available"
```

### Output has the same idea
The monitor also needs:
```
data register
status register
```

For the LC-3:
```
DDR = Display Data Register
DSR = Display Status Register
```

with:
```
DDR = xFE06
DSR = xFE04
```

Again:
```
DDR → data to display
DSR → status of display
```

The DSR's bit 15 is the ready bit.

### DSR ready bit
Suppose:
```
DSR[15] = 0
```

That means:

    The monitor is busy processing the previous character.

So don't send another character yet.

When:
```
DSR[15] = 1
```

the processor can write the next character to DDR.

Therefore:
```
DSR[15] = 1
       ↓
monitor ready
       ↓
write DDR
       ↓
monitor becomes busy
       ↓
DSR[15] = 0
```

Again we have synchronization.