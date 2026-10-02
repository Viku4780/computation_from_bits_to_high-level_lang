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