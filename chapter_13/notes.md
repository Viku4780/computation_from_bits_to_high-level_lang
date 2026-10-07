## What is a "control structure"?
let's start with problem.

Suppose i write:
```
int x = 10;

printf("A\n");
printf("B\n");
printf("C\n");
```

The computer basically does:
```
A
 ↓
B
 ↓
C
```

This is sequential execution.

The CPU executes one instructions, then another, then another.

But real programs need much more than that.

We need to say things like:

    If the user entered the correct password, allow access.

or:

    Keep asking until the user enters a valid number.

or:

    Do this 100 times.

or:

    if the operation is +, add. if it is -, subtract.

This means the program cannot simply move straight downward.

it needs to change the path of execution.

That is the fundamental idea behind control structure.


### The core abstraction

#### 1. Selection
Choose which path to execute.

#### 2. Iteration
Choosee whether to execute a path again.


### Control structures are really branches
When you write:
```
if (x < 10){
    printf("small");
}
```

The CPU doesn't have an instruction called:
```
IF
```

at the fundamental machine level.

Instead, the compiler turns your high-level idea into something resembling:
```
calculate x < 10
       |
       v
   condition?
    /     \
 false    true
  |         |
  |         v
  |       execute body
  |         |
  └─────────┘
```

At the machine level, this becomes comparisons + conditional branches/jumps.

This is the bridge:
```
C
│
├── if
├── while
├── for
├── do-while
├── switch
├── break
└── continue
       ↓
control flow
       ↓
conditional/unconditional branches
       ↓
machine instructions
       ↓
CPU changes PC
       ↓
different instruction executes
```

Remember your earlier understanding of the program counter (PC).

The PC normally says:

    Which instruction should execute next?

A control structure essentially changes the answer to:

    Which instruction should execute next?

That is the essence