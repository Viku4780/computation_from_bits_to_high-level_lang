The main point of this chapter is:

    How do we organize larger, more complicated information and larger programs?


```
                 CHAPTER 8
                     |
        +------------+------------+
        |                         |
   SUBROUTINES               DATA STRUCTURES
        |                         |
   call / return         +--------+--------+---------+
        |                |        |        |         |
   save registers      STACK    QUEUE   STRING   ...
        |
     enables
        |
    RECURSION
        |
      uses
      STACK
```

## What is a data structure?
Suppose i give you these values:
```
10
25
42
7
91
```

Those are just individual values.

But real programs often need to represent something more organized:
```
Employee
 ├── name
 ├── salary
 ├── age
 └── department
```

or:
```
People waiting in line
A → B → C → D
```

or:
```
Browser history
Page1 → Page2 → Page3
```

or:
```
Characters of a name
B i l l   L i n v i l l
```

the problem is no longer simply:

    How do i store a value?

it becomes:

    How should multiple values be organized, and what operations should be allowed on them?

That is where data structure enter.


### Abstract Data type

    An abstract data type is defined by what you can do with it, not by how it is physically implemented.

Let's take a stack

A stack says:
```
Last thing inserted
        ↓
    removed first
```

That rule is called:
#### LIFO - Last In, First Out

Suppose we do:
```
PUSH A
PUSH B
PUSH C
```

The stack is conceptually:
```
TOP
 ↓
 C
 B
 A
```

Then:
```
POP
```

gives:
```
C
```

Another:
```
POP
```

gives:
```
B
```

Important part is that the stack doesn't care whether it is implemented using:
```
memory
registers
an array
some hardware structure
```

The behavior is what defines the stack.

that is the meaning of the abstraction.


### Subroutines:
you have already seen assembly instruction like:
```
ADD
LDR
BR
STR
```
But imagine your program contains the same 20 instructions in five different places.

That would be ugly:
```
code
code
code
code
code

same code again

code

same code again

code
```

instead, we can write it once:
```
SUBROUTINE_A
    instructions...
    instructions...
    instructions...
    return
```

and jump to it whenever we need it.

this is the basic idea of a subroutine.

in C, the equivalent concept is a function.

for example:
```
void printHello(void)
{
    printf("Hello\n");
}
```

then:
```
printHello();
printHello();
printHello();
```

the code for the operation exists only once


### Caller and callee
Suppose:
```
MAIN
  |
  | calls
  ↓
FUNCTION
```

The program doing the calling is the:
#### caller

The program being called is the:
#### callee

So:
```
MAIN  --------calls-------->  SUBROUTINE
caller                         callee
```

This terminology becomes extremely useful later when you study function calls, stack frames, calling conventions, ABI rules, and operating systems.

### The call/return mechanism
Suppose our program is here:
```
MAIN:

    instruction A
    instruction B
    JSR SUBROUTINE
    instruction C
    instruction D
```

When we execute:
```
JSR SUBROUTINE
```

we need two things to happen

#### Thing 1 - Go to the subroutine
The PC must become the address of:
```
SUBROUTINE
```
So execution changes:
```
MAIN
 ↓
JSR
 ↓
SUBROUTINE
```

#### Thing 2 - remember where to come back
Suppose:
```
1000   instruction A
1001   JSR SUBROUTINE
1002   instruction C
1003   instruction D
```

when we execute:
```
JSR SUBROUTINE
```

the computer must remember:
```
1002
```
because that's where it needs to continue after the subroutine

The chapter calls this the return linkage.

in the LC-3, the return linkage is stored in:
```
R7
```

so conceptaully:
```
Before JSR:

PC = 1001
R7 = whatever
```

After JSR:
```
PC = address of SUBROUTINE
R7 = 1002
```

The book specifically defines JSR(R) as doing these two jobs: loading the PC with the subroutine address and loading R7 with the address immediately after the call.

### Returning from the subroutine
At the end of the subroutine we have:
```
JMP R7
```

Remember:
```
R7 = return address
```

Therefore:
```
JMP R7
```

means:

    Put the value contained in R7 into the PC.

So:
```
MAIN
  |
  | JSR
  ↓
SUBROUTINE
  |
  | JMP R7
  ↓
MAIN continues
```

That is the complete fundamental call/return mechanism.

### The most important problem: R7 can be destroyed
Now comes the first real difficulty.

Suppose:
```
MAIN
 |
 | JSR A
 ↓
A
 |
 | JSR B
 ↓
B
```

Remember what happens during JSR:
```
R7 = return address
```

So when MAIN calls A:
```
R7 = address back to MAIN
```

But then A calls B:
```
JSR B
```

Now:
```
R7 = address back to A
```

The old value:
```
address back to MAIN
```

has been overwritten.

So the machine has forgotten how to return from A to MAIN.

This is a fundamental problem.

### Why recursion makes this problem worse
Suppose:
```
FACT:
    ...
    JSR FACT
    ...
    RET
```

Now the function is calling itself.

Imagine:
```
FACT #1
   |
   | JSR FACT
   ↓
FACT #2
   |
   | JSR FACT
   ↓
FACT #3
```

Every call changes R7.

Therefore:
```
FACT #1's return address
```

gets destroyed by:
```
FACT #2
```

and then FACT #2's return address gets destroyed by FACT #3.

So we need somewhere to store these return addresses.

And that leads directly to:
## THE STACK