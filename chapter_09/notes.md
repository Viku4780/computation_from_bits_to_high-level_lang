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


### One subtle question
You might ask:

    "But the HALT routine ends with RTI. How can it return if the machine is already halted?"

Exactly.

The important thing is:
```
clearing RUN
```

stops the machine from continuing the instruction cycle.

So the later RTI instruction is never actually executed before the machine stops.


## Now we reach the biggest concept: interrupts
Polling was:
```
CPU → "Are you ready?"
CPU → "Are you ready?"
CPU → "Are you ready?"
```

Interrupt-driven I/O reverses the relationship.

Instead:
```
CPU:
I'm doing useful work.

Keyboard:
"HEY! I HAVE INPUT!"

CPU:
Okay, stop what you're doing temporarily.
I'll handle it.
```

This is an:
Interrupt

### Why interrupts are useful
Suppose:
```
CPU = extremely fast
keyboard = extremely slow
```

With polling:
```
CPU waits
CPU waits
CPU waits
CPU waits
```

With interrupts:
```
CPU computes other things
CPU computes other things
CPU computes other things
keyboard interrupts
CPU reads character
CPU goes back
```

So the CPU doesn't waste most of its time repeatedly checking.


### But an interrupt doesn't magically happen
The chapter gives three necessary ideas.

The device must:
```
1. want service
2. have permission to request service
3. have sufficient priority
```

#### Condition 1 — device wants service
For the keyboard:
```
someone typed a key
```

therefore:
```
KBSR[15] = 1
```

For the monitor:
```
previous character finished
```

therefore:
```
DSR[15] = 1
```

So the ready bit tells us:

    "I need / can accept service."


#### Condition 2 — interrupt enable
The device also needs permission to interrupt the processor.

The LC-3 uses:
```
IE = Interrupt Enable
```

For the device status registers:
```
bit 14 = IE
bit 15 = ready
```

So conceptually:
```
Interrupt Request
      =
IE AND Ready
```

Meaning:
```
IE = 0
    ↓
device cannot interrupt

IE = 1
Ready = 0
    ↓
no interrupt

IE = 1
Ready = 1
    ↓
interrupt request
```

The chapter explicitly presents the request as the logical AND of the interrupt-enable and ready bits.


#### Condition 3 — priority
Suppose:
```
current program = PL2
keyboard interrupt = PL4
```

Then:
```
4 > 2
```

So the keyboard can interrupt the program.

But imagine:
```
current program = PL6
keyboard = PL4
```

Then:
```
4 < 6
```

The keyboard should not interrupt the current program in this model.

So:
```
request priority > current priority
```

is required.


### What if several devices want service?
Imagine:
```
Keyboard → PL4
Timer    → PL5
Emergency → PL7
```

all request service simultaneously.

The processor shouldn't randomly choose one.

The LC-3 uses a:
## Priority encoder

Conceptually:
```
PL4 ─┐
PL5 ─┼──→ priority encoder → PL7
PL7 ─┘
```
The highest-priority request wins.

### The INT signal
The final question is:

    "Should the processor actually stop its current program?"

The logic decides:
```
highest interrupt request
          ↓
compare with current program priority
          ↓
higher?
   yes       no
    ↓         ↓
 INT=1       normal execution
```

So the processor only enters interrupt handling if the request is sufficiently urgent.


### Interrupts happen at instruction boundaries
This is one of the most important implementation details.

Imagine the CPU is halfway through:
```
ADD R1,R2,R3
```

and suddenly an interrupt appears.

Should it stop:
```
halfway through ADD?
```

No.

That would make the processor state messy.

Instead, the LC-3 waits until the current instruction has completed.

So:
```
instruction A
      ↓
instruction finishes
      ↓
check INT
      ↓
interrupt? 
   /      \
 yes       no
 ↓          ↓
ISR      next instruction
```

The chapter emphasizes that interrupts are asynchronous to the processor's synchronous instruction cycle, but are recognized at an instruction boundary.


### Why instruction boundaries make everything easier
Imagine:
```
instruction = half finished
```

If an interrupt happens there, the system would need to remember:
```
which phase?
which micro-operation?
which temporary value?
what remains?
```

Instead:
```
instruction completed
```

means:

    The processor is in a clean, well-defined state.

Then save that state.

Much easier.

### Now let's trace an interrupt completely
Suppose:
```
User program A
      |
      | executing
      ↓
instruction x3006
```

Keyboard interrupt occurs.

We will follow the machine.

#### Stage 1 — finish the current instruction
The processor finishes:
```
x3006
```

Then tests:
```
INT?
```

Suppose:
```
INT = 1
```

Now the processor does not fetch x3007 normally.

Instead it enters interrupt processing.


#### Stage 2 — save interrupted state
The LC-3 needs to preserve:
```
PC
PSR
```

The saved PC is the address of the instruction that would have executed next.

So:
```
current instruction = x3006

saved PC = x3007
```

The PSR contains things such as:
```
User/Supervisor
priority
N/Z/P
```

These go onto the supervisor stack.

If we were in User mode, the LC-3 first switches stack context:
```
R6
 ↓
supervisor stack
```

The user stack pointer is saved separately.


#### Stage 3 — load interrupt service routine
Now the interrupting device provides an:
```
8-bit interrupt vector
```

Let's imagine:
```
INTV = x80
```

The processor expands this to:
```
x0180
```

because the Interrupt Vector Table occupies:
```
x0100 – x01FF
```

Then:
```
memory[x0180]
```

contains the address of the keyboard interrupt service routine.

Suppose:
```
memory[x0180] = x6200
```

Then:
```
PC = x6200
```

Now the interrupt handler starts running.


```
Trap
  ↓
x0000-x00FF

Interrupt
  ↓
x0100-x01FF
```

### What PSR does the interrupt service routine get?
The interrupt service routine must be privileged.

So:
```
PSR[15] = 0
```

and its priority becomes the priority associated with the interrupt.

The chapter also initializes the condition-code bits for the service routine because no service-routine instruction has executed yet.

The important mental model is:
```
old PSR
   ↓
saved

new PSR
   ↓
privileged + interrupt's priority
```

#### Stage 4 — service the interrupt
Now:
```
PC = ISR address
```

So the processor simply starts executing instructions there.

For a keyboard ISR, it may:
```
read KBDR
store/process character
update data
```

and then eventually:
```
RTI
```

#### Stage 5 — RTI
Now the supervisor stack contains something like:
```
saved PC
saved PSR
```

The ISR executes:
```
RTI
```

The processor restores:
```
PC
PSR
```

So:
```
PC = x3007
PSR = old user-program PSR
```

Then, because the old program was in User mode, the processor switches R6 back to the user stack.

Finally:
```
fetch x3007
```
and the user program continues.

```
|                           | TRAP                       | Interrupt                 |
| ------------------------- | -------------------------- | ------------------------- |
| Who initiates?            | Current program            | External event/device     |
| Why?                      | Request a service          | Demand attention          |
| Synchronous with program? | Yes, caused by instruction | Asynchronous              |
| Vector comes from         | TRAP instruction           | Interrupting event/device |
| Privilege change          | Yes                        | Yes                       |
| Save state                | PC + PSR                   | PC + PSR                  |
| Return                    | RTI                        | RTI                       |
```

### The nested interrupt example
```
Program A
    ↓
Device B interrupt
    ↓
B service routine
    ↓
Device C higher-priority interrupt
    ↓
C service routine
    ↓
RTI
    ↓
B resumes
    ↓
RTI
    ↓
A resumes
```

### Interrupts aren't only about I/O

An interrupt can represent things such as:
```
timer
machine check
power failure
```

So don't permanently equate:
```
interrupt = keyboard
```

That's only one example.

The broader idea is:

    An external event needs processor attention.

### Polling has one more subtle problem

Suppose you're doing:
```
LDI R1,DSR
BRzp LOOP
STI R0,DDR
```

You might think:

    "This is one operation: check whether display is ready, then write."

But at the hardware level, these are three separate instructions.

What if an interrupt happens between them?

### The race-like problem
Suppose:
```
Step 1:
LDI
```

reports:
```
DSR ready = 1
```

Then:
```
Step 2:
BRzp
```

does not branch.

Before:
```
Step 3:
STI
```

an interrupt occurs.

The interrupt service routine writes something to the display.

Now the display may become:
```
busy
```

The original program returns and executes:
```
STI R0,DDR
```

But the display is no longer ready.

So the earlier observation:
```
ready = 1
```

is now stale.

The chapter gives a concrete example in which this can cause one expected character to be lost


### What did we assume incorrectly?
We treated:
```
LDI
BR
STI
```

as though it were one indivisible operation.

But it isn't.

It's three instructions.

An interrupt can occur between them.

So we need a way to protect the critical sequence.


### The solution: temporarily disable interrupts
The book's solution is interesting.

It does not say:

    disable interrupts for the entire polling operation.

That would be bad.

Instead:
```
enable interrupts
        ↓
check whether device is ready
        ↓
not ready?
        ↓
enable again / continue polling

ready?
        ↓
temporarily disable interrupts
        ↓
perform critical sequence
        ↓
restore previous PSR
```

The key is:

    Interrupts are disabled only around the tiny critical section.

### This is a critical-section idea
You are going to see this concept again and again in your embedded/RTOS work.

A critical section is a small piece of code that must not be interrupted in a way that would break its assumptions.

Conceptually:
```
normal code
     ↓
critical section
     ↓
normal code
```

### The deeper story: synchronization
There is also a second theme.

The CPU and device operate at different times.

So we need:
```
status bits
ready bits
polling
interrupts
```

All of these solve the same broad problem:

    How do two independently operating components coordinate?

You can see the progression:
```
DEVICE
   ↓
ready bit
   ↓
CPU checks
   ↓
polling
```

or:
```
DEVICE
   ↓
ready bit + interrupt enable
   ↓
interrupt request
   ↓
CPU reacts
```

```
| Polling                            | Interrupt-driven                                        |
| ---------------------------------- | ------------------------------------------------------- |
| CPU repeatedly checks              | Device signals CPU                                      |
| CPU controls interaction           | Device initiates interaction                            |
| Can waste CPU time                 | CPU can do useful work                                  |
| Simple                             | More complex                                            |
| No interrupt mechanism required    | Requires interrupt mechanism                            |
| Waiting happens explicitly in code | Waiting can happen implicitly while other work executes |
```

### A full example from start to finish
Let's take:
```
TRAP x23
```

and trace it as a real story.

Before
```
Mode = User
PC = address of TRAP
R6 = user stack pointer
```

TRAP executes
```
save PC
save PSR
switch to supervisor stack
switch to Supervisor mode
```

Vector lookup
```
x23
 ↓
x0023
 ↓
memory[x0023]
 ↓
x04A0
```

OS
```
PC = x04A0
```

Now the keyboard service routine executes.

Result
```
R0 = ASCII character
```

Finish
```
RTI
```

RTI
```
restore PC
restore PSR
restore appropriate stack
```

Continue
```
PC = instruction after TRAP
```

From the user program's point of view:
```
TRAP
 ↓
character magically appears in R0
 ↓
next instruction
```

But now you know it isn't magic.

### Full interrupt example
Now:
```
User program A
```

is executing.

Keyboard does:
```
KBSR[15] = 1
```

and:
```
KBSR[14] = 1
```

so:
```
ready AND enabled = 1
```

Suppose keyboard priority:
```
PL4
```

and current program:
```
PL2
```

Therefore:
```
4 > 2
```

so the processor asserts:
```
INT
```

At the instruction boundary:
```
finish current instruction
        ↓
save PC + PSR
        ↓
switch to supervisor stack
        ↓
interrupt vector
        ↓
interrupt vector table
        ↓
ISR address
        ↓
execute ISR
        ↓
RTI
        ↓
restore PC + PSR
        ↓
resume user program
```

That is the entire mechanism.