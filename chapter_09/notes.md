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


## Asynchronous vs synchronous

#### Synchronous
Two things operate in a coordinated, predictable relationship.

For example, imagine:
```
device produces one item
every exact 100 cycles
```

The processor knows when it will happen.

No uncertainty.

#### Asynchronous
The events happen independently.

For example:
```
CPU: running continuously

human:
     types a key...
           sometime...
                 maybe now...
```

The processor doesn't know exactly when the key will arrive.

That's asynchronous.

The chapter emphasizes that most processor/I/O interaction is asynchronous because devices operate at different speeds and are not locked to the processor's timing.

### The ready bit is a simple handshake
Think of the ready bit as:
```
DEVICE:
"DATA IS READY"

CPU:
"Okay, I'll read it."
```

Then:
```
DEVICE:
"DATA IS NOT READY"
```

This tiny bit creates a synchronization protocol.

It's primitive, but extremely important.


## Memory-mapped I/O
The LC-3 doesn't need special:
```
READ_KEYBOARD
WRITE_MONITOR
```

instructions.

Instead, it gives hardware registers addresses.

For example:
```
xFE00 → KBSR
xFE02 → KBDR
xFE04 → DSR
xFE06 → DDR
```

This means the same load/store machinery used for memory can interact with hardware.

This is:
Memory-mapped I/O

### Why is it called memory-mapped?
Because from the instruction's perspective:
```
LDR
STR
LD
ST
```
are still being used.

The address tells the hardware what resource you're accessing.

So conceptually:
```
normal memory address
        ↓
     memory

I/O address
        ↓
     device
```
The address space is being used for both.

### Very important distinction
This:
```
x3000
```
may refer to actual memory.

But:
```
xFE02
```

does not mean:

    "ordinary RAM location xFE02"

In the LC-3 it means:

    the keyboard data register.

So:
```
address
   ↓
address-control logic
   ↓
memory OR device register
```

This is why the processor can use familiar load/store instructions for I/O.

### How a memory-mapped input works internally
Suppose we do:
```
LDI R0,KBDR
```

The important conceptual steps are:
```
1. obtain the target address
2. place address in MAR
3. address-control logic sees xFE02
4. instead of selecting normal memory,
   it selects KBDR
5. KBDR → MDR
6. MDR → R0
```

So:
```
KBDR
  ↓
MDR
  ↓
R0
```

The book explains that memory-mapped I/O reuses the same basic data-movement pathway as memory, with the address-control logic selecting the appropriate device register.


## Polling

Suppose the CPU wants keyboard input.

It can do:
```
START
    LDI R1,KBSR
    BRzp START
    LDI R0,KBDR
```

Conceptually:
```
check ready?
    ↓
 NO ───────→ check again
    |
   YES
    ↓
read character
```

That's polling.

The processor keeps asking:

    "Are you ready yet?"


### The problem with polling
The CPU is doing:
```
LDI
BR
LDI
BR
LDI
BR
...
```

while the keyboard may take a long time to produce the next character.

So the CPU spends time doing essentially nothing useful.

This is called spinning or busy-waiting.

The chapter's example shows that a processor can spend huge amounts of time waiting for human input when polling is used.

now this motivates:
## Interrupts

### Before interrupts: TRAP
We've already used things like:
```
TRAP x23
TRAP x21
TRAP x22
TRAP x25
```

But until now, you didn't know the complete story.

A TRAP is essentially a request:

    "Operating system, please perform this service for me."

The chapter notes that the generic term for this is a system call/service call.


## TRAP as a controlled doorway
Think of a TRAP like:
```
USER MODE
    |
    | TRAP
    ↓
[ controlled doorway ]
    |
    ↓
SUPERVISOR MODE
    |
    ↓
OS service routine
```

The user program does not simply jump anywhere it wants.

The processor follows a controlled trap mechanism.

This is the beginning of the OS security/protection model you were asking about in your OS studies.


### Trap vectors
The instruction:
```
TRAP x23
```

contains:
```
trap vector = x23
```

The vector identifies which service routine is wanted.

The LC-3 has a Trap Vector Table in:
```
x0000 – x00FF
```

Each entry contains the starting address of a trap service routine.

For example, according to the chapter:
```
x0021 → address of character output routine
x0023 → address of keyboard input routine
x0025 → address of halt routine
```

The corresponding service routines in the example start at:
```
x0420
x04A0
x0520
```

respectively.


### Let's trace TRAP x23 completely
Suppose:
```
TRAP x23
```

is executing in a user program.

#### Step 1 — PC has already been incremented
As part of the instruction cycle, the PC has advanced.

So the PC now points to:
```
instruction immediately after TRAP
```

This is important because that's where we need to return.

#### Step 2 — Save state
The LC-3 needs enough information to later resume the interrupted/calling program.

It saves:
```
PSR
PC
```
on the supervisor stack.

If currently in User mode, the processor first switches the active stack from the user stack to the supervisor stack.

```
user program:

instruction A
TRAP x23
instruction B
instruction C
```
saved PC:
```
address of instruction B
```


```
before TRAP
    ↓
save PSR
    ↓
OS executes
    ↓
restore PSR
    ↓
user program resumes
```

#### Step 3 — switch to Supervisor mode
The LC-3 sets:
```
PSR[15] = 0
```

So the service routine can access privileged resources.

The priority stays at the priority of the calling program for a TRAP according to the chapter's mechanism

#### Step 4 — use the trap vector
The vector:
```
x23
```

is zero-extended to:
```
x0023
```

Then:
```
memory[x0023]
```

contains:
```
x04A0
```

So:
```
PC = x04A0
```

And now the processor begins executing the keyboard input service routine.

```
User program
     |
     | TRAP x23
     ↓
save PSR
save return PC
     ↓
switch to supervisor
     ↓
x23 → x0023
     ↓
memory[x0023] = x04A0
     ↓
PC = x04A0
     ↓
OS keyboard service routine
     ↓
read keyboard
     ↓
put result in R0
```


## RTI — Return from Trap or Interrupt
Now the OS is finished.

How does it return?

Not:
```
BR
```

Not ordinary:
```
JMP
```

Instead:
```
RTI
```

Why?

Because the processor needs to restore both:
```
PC
PSR
```

The RTI instruction pops the saved state from the supervisor stack.


### RTI step-by-step
Conceptually:
```
Supervisor stack:

saved PC
saved PSR
```

RTI does:
```
pop → PC
pop → PSR
```

Then:
```
processor state = old state
```

If that old state indicates User mode, the processor switches back to the user stack as well.

Therefore:
```
OS
 ↓
RTI
 ↓
restore PC
restore PSR
restore stack context
 ↓
user program continues
```


### A huge conceptual distinction
A normal subroutine is basically:
```
program → program
```

A system call is:
```
user program → operating system
```

And because that crosses a privilege boundary, the machine needs stronger protection and state handling.

This is exactly the OS concept you were studying earlier:
```
system call
=
controlled transition from user execution into privileged OS execution
```

The LC-3 TRAP mechanism is the textbook's simple concrete model of that idea.


### PUTS — an actual OS service
The chapter then builds a string-output service:
```
TRAP x22
```

called:
```
PUTS
```

The user program provides:
```
R0 = address of string
```

The OS routine then:
```
read character
check if zero
wait for display
send character
advance pointer
repeat
```

This is an excellent example of abstraction.

User program says:
```
"print this string"
```

OS handles:
```
hardware details
status checking
character-by-character output
```

The chapter's PUTS routine is specifically a service for writing a null-terminated string to the console.

### The HALT service routine
The LC-3 doesn't need a special HALT opcode in this design.

Instead:
```
TRAP x25
```

invokes an OS halt routine.

That routine eventually clears:
```
MCR[15]
```

where MCR is:
```
Master Control Register
```

at:
```
xFFFE
```

The top bit controls the RUN latch.

Clearing it stops the clock.