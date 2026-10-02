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