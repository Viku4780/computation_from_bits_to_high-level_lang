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

### Saving and restoring registers
Before we get fully into the stack, there is another major problem with subroutines.

Suppose the caller has:
```
R1 = 100
R2 = 200
R3 = 300
```

Then it calls a subroutine.

Inside the subroutine:
```
LD R1,...
LD R2,...
LD R3,...
```

Now:
```
R1 = new value
R2 = new value
R3 = new value
```

The caller's values are gone.

But what if the caller expected them to still exist?

We have a problem.

### Save and restore
The basic solution is extremely simple.

Before destroying a register:
```
save it
```

Then after you're finished:
```
restore it
```

Conceptually:
```
R1 = 100

       ↓

save 100 in memory

       ↓

R1 used by subroutine

       ↓

restore 100

       ↓

R1 = 100 again
```

### Caller save
Suppose the caller knows that some operation will destroy a register.

Then the caller can save it.

```
CALLER

save R7
JSR SUBROUTINE
restore R7
```

That's:

caller save

The caller handles the preservation.

### Callee save
Suppose the callee knows:

    "I need R1 and R2 for my own work."

Then the subroutine itself saves them:
```
SUBROUTINE

save R1
save R2

do work

restore R2
restore R1

return
```

That's:

#### callee save

The book's key reasoning is very good:

    The callee knows which registers it needs.

Therefore it makes sense for the callee to save those registers.

### The general rule
You should build this mental rule:

    If a value will be destroyed but you still need that value later, save it before destruction and restore it before using it again.

This is broader than registers.

It applies to:
```
register values
return addresses
local variables
function state
recursive state
temporary computation values
```

And that is why the stack becomes so useful.

## JSR vs JSRR
The LC-3 has two forms.

#### JSR
```
JSR LABEL
```

uses PC-relative addressing.

The target address is calculated roughly as:
```
target = incremented PC + sign-extended offset
```

The book gives JSR an 11-bit offset.

#### JSRR
```
JSRR R5
```

uses a register containing the subroutine address.

Conceptually:
```
R5 = 0x3002
```

then:
```
JSRR R5
```

means:
```
PC = R5
```

while also placing the return address in R7.

So:
```
JSR
    target comes from instruction's offset

JSRR
    target comes from a register
```

This is useful when the address isn't conveniently encoded as a nearby PC-relative target.


### Library routines
Now the chapter gives us another powerful idea.

Suppose you need square root.

You could write a square-root algorithm yourself.

But you probably don't want to.

Instead:
```
Your program
     |
     | calls
     ↓
SQRT library routine
```

You just need to know the interface:
```
Input:
    where do I put x?

Output:
    where do I get sqrt(x)?
```

You don't necessarily need to know how the library internally calculates it.

This is a practical example of abstraction.

The chapter's example uses a square-root library routine and then connects it to separate object modules combined by the linker.

This is directly connected to what you learned earlier about:
```
source files
   ↓
object files
   ↓
linking
   ↓
executable
```

## Now: THE STACK
Forget implementation for one minute.

A stack has one defining rule:
```
LIFO

Last In, First Out

Imagine a pile of plates.

      ┌─────┐
      │  C  │ ← remove first
      ├─────┤
      │  B  │
      ├─────┤
      │  A  │
      └─────┘
```

You can't conveniently remove A first.

You have to remove:
```
C
then B
then A
```

That is a stack.

### PUSH and POP
Two basic operations:

#### PUSH
Put an item onto the stack.
```
PUSH A
```

#### POP
Take the top item off.
```
POP
```

Example:
```
PUSH A
PUSH B
PUSH C
```

Stack:
```
C ← top
B
A
```

Then:
```
POP → C
POP → B
POP → A
```

That's LIFO.

The book explicitly treats the stack as an abstract data type whose definition comes from its access rules, not its implementation.

### Two different implementations
The same abstract stack can have different implementations.

The book shows:

#### Implementation 1 — hardware registers
The values themselves move.

For example:
```
Before:

[18]
[31]
[ 5]
[12]
```

After a pop, data may physically shift.

#### Implementation 2 — memory
The values don't move.

Instead:

    We move a pointer that tells us where the top is.

That's a giant idea.

### Stack implemented in memory
The LC-3 implementation uses:
```
R6
```

as the stack pointer.

So:
```
R6 = address of top of stack
```

The book's example reserves:
```
x3FFF
x3FFE
x3FFD
x3FFC
x3FFB
```

and the stack grows toward lower addresses.

This gives:
```
higher addresses

x3FFF
x3FFE
x3FFD
x3FFC
x3FFB

lower addresses
```
The phrase to remember:

    The LC-3 stack in this example grows toward zero.

### Empty stack
Initially:
```
R6 = x4000
```

Why x4000?

Because the stack's available storage is:
```
x3FFF ... x3FFB
```

So x4000 represents:

    just beyond the top end of the reserved area

Conceptually:
```
x4000   ← R6
----------------
x3FFF
x3FFE
x3FFD
x3FFC
x3FFB
```

No actual stack item exists yet.

### PUSH in memory
The book gives:
```
ADD R6,R6,#-1
STR R0,R6,#0
```

Let's slow this down.

Suppose:
```
R6 = x4000
R0 = 18
```

First:
```
ADD R6,R6,#-1
```

becomes:
```
R6 = x3FFF
```

Then:
```
STR R0,R6,#0
```

stores:
```
18
```

at:
```
x3FFF
```

So:
```
R6 → x3FFF
      ↓
     18
```

That's PUSH.

### PUSH another value
Suppose:
```
R6 = x3FFF
```

and we want:
```
31
```

First:
```
ADD R6,R6,#-1
```

gives:
```
R6 = x3FFE
```

Then:
```
STR R0,R6,#0
```

stores 31:
```
x3FFF → 18
x3FFE → 31 ← R6
```

So the top is 31.

Then:
```
PUSH 5
```

gives:
```
x3FFF → 18
x3FFE → 31
x3FFD → 5 ← top
```

### POP
The book uses:
```
LDR R0,R6,#0
ADD R6,R6,#1
```

Again, slowly.

Suppose:
```
R6 = x3FFD
```

and:
```
x3FFD = 5
```

First:
```
LDR R0,R6,#0
```

means:
```
R0 = memory[x3FFD]
```

so:
```
R0 = 5
```

Then:
```
ADD R6,R6,#1
```

makes:
```
R6 = x3FFE
```

So 5 has logically been removed.

### Important: POP does NOT erase memory
This is a very important memory concept.

Suppose:
```
x3FFD = 5
```

and then we POP it.

Memory may still physically contain:
```
x3FFD = 5
```

But the stack pointer has moved.

Therefore the stack no longer considers x3FFD to contain a stack element.

This distinction is crucial:
```
physical memory contents
        ≠
logical data structure contents
```

The book emphasizes this point for both stacks and queues.

This is a beautiful example of abstraction.

### Physical memory vs logical structure
Imagine:
```
Memory:

x3FFF → 18
x3FFE → 31
x3FFD → 5
x3FFC → 12
```

Suppose R6 now says:
```
R6 = x3FFE
```

Then logically the stack is:
```
31
18
```

Even though memory still contains:
```
5
12
```

So:
```
Memory:
18 31 5 12

Stack:
18 31
```

The stack protocol determines what is accessible as stack data.

That's abstraction in action.

### Stack protocol
The book calls the rules governing access the stack protocol.

Conceptually:
```
Correct:

PUSH → changes top
POP  → removes top

Incorrect:

randomly read old memory
randomly write inside stack
```

Why?

Because if you break the protocol, the structure is no longer behaving like a stack.

### Underflow
What if:
```
stack = empty
```

and you do:
```
POP
```

There is nothing to remove.

That's:

#### UNDERFLOW
Think:
```
too little data
```

The book checks for this by testing whether:
```
R6 = x4000
```

which means the stack is empty.

### Overflow
Now the opposite.

Suppose the stack has:
```
x3FFF
x3FFE
x3FFD
x3FFC
x3FFB
```

all occupied.

Now another PUSH happens.

There is nowhere to put it.

That's:

#### OVERFLOW
Think:
```
too much data
```

### Success/failure reporting
The chapter's PUSH and POP routines use:
```
R5 = 0 → success
R5 = 1 → failure
```

So a caller can do:
```
call PUSH
check R5
```

and know whether insertion succeeded.

This is another good example of an interface:
```
Input:
    R0 = value

Output:
    R5 = success/failure
```

The caller doesn't need to know every internal instruction.

---

```
              R6
              ↓
      ┌─────────────┐
x3FFF │             │
x3FFE │             │
x3FFD │    STACK    │
x3FFC │             │
x3FFB │             │
      └─────────────┘

R6 = top of stack
```

Push:
```
R6--
store value
```

Pop:
```
read value
R6++
```

And therefore:
```
PUSH → address goes downward
POP  → address goes upward
```

## Recursion

    a function expressed in terms of itself.

for example:
```
FACT(n) = n x FACT(n-1)
```

That means:
```
FACT(5)
= 5 × FACT(4)

= 5 × 4 × FACT(3)

= 5 × 4 × 3 × FACT(2)

= 5 × 4 × 3 × 2 × FACT(1)

= 5 × 4 × 3 × 2 × 1
```

The key is:
### Every recursive problem needs a base case.

### Base case
For factorial:
```
1! = 1
```

Therefore:
```
if n == 1
    stop recursion
```

Without that:
```
FACT(5)
 ↓
FACT(4)
 ↓
FACT(3)
 ↓
FACT(2)
 ↓
FACT(1)
 ↓
FACT(0)
 ↓
FACT(-1)
 ↓
FACT(-2)
 ↓
...
```

Forever.

So mentally:
```
recursive case
      ↓
smaller problem
      ↓
smaller problem
      ↓
...
      ↓
base case
```

### Why recursion requires a stack
This is the real essence.

Imagine:
```
FACT(5)
```

It calls:
```
FACT(4)
```

But FACT(5) isn't finished yet.

Then FACT(4) calls:
```
FACT(3)
```

FACT(4) isn't finished either.

Then:
```
FACT(2)
```

Then:
```
FACT(1)
```

Now we have:
```
FACT(5) waiting
FACT(4) waiting
FACT(3) waiting
FACT(2) waiting
FACT(1) running
```

Each waiting invocation needs its own information:
```
original n
return address
temporary values
register state
```

That's exactly what the stack is good at.

### Think of recursive calls as suspended worlds
When:
```
FACT(5)
```

calls:
```
FACT(4)
```

FACT(5) hasn't disappeared.

It is simply paused.

Think:
```
FACT(5)
┌─────────────────────┐
│ "I'm waiting for    │
│ FACT(4) to finish"  │
└─────────────────────┘
          ↓
FACT(4)
┌─────────────────────┐
│ "I'm waiting for    │
│ FACT(3) to finish"  │
└─────────────────────┘
          ↓
FACT(3)
```

The stack stores these waiting contexts.

That is the real relationship:
```
recursion
   ↓
nested calls
   ↓
many unfinished calls
   ↓
stack stores their state
```

### Factorial: why the naive recursive version is ugly at machine level
Suppose:
```
FACT(n)
```

needs:
```
n
return address
register state
```

And then it recursively calls:
```
FACT(n-1)
```

Now another copy of those things is needed.

So for:
```
FACT(5)
```

you might conceptually get:
```
Stack

FACT(1) state
FACT(2) state
FACT(3) state
FACT(4) state
FACT(5) state
```

As recursion gets deeper, the stack grows.

The book specifically shows that return linkages, the original n, and register values must be pushed so each recursive invocation can later recover its own state.


### Why factorial is a bad example of recursion
This is subtle.

The formula is beautiful:
```
n! = n × (n-1)!
```

But the iterative solution is extremely simple.

You can simply do:
```
result = 1

for i = 2 to n
    result = result * i
```

No recursive stack growth.

No repeated call overhead.

So for factorial:
```
recursive:
beautiful mathematical expression
+
extra execution overhead

iterative:
slightly less elegant
+
simple and efficient
```

### So when is recursion actually good?
This is where the chapter's maze example becomes important.

Imagine a maze:
```
+---+---+---+
| S     |   |
+   +   +   +
|   |       |
+   +---+   +
|       | E |
+---+---+---+
```

At each cell you may have:
```
north
east
south
west
```

You don't know which path will eventually lead to the exit.

You can try one direction.

If it fails:
```
come back
try another
```

That naturally creates a recursive search.

### The maze algorithm
At the current cell:

#### Step 1
Ask:
```
Is there an exit here?
```

If yes:
```
return YES
```

#### Step 2
Mark this cell as visited.

The book calls this a breadcrumb.

Why?

Because otherwise you could get:
```
A → B → C → A → B → C → A → ...
```

forever.

#### Step 3
Try north.

If there is a door and the cell has not been visited:
```
FIND_EXIT(north)
```

#### Step 4
If north fails:
```
try east
```

Then:
```
south
```

Then:
```
west
```

#### Step 5
If all possibilities fail:
```
return NO
```

This is a beautiful use of recursion.

### Why recursion works naturally for the maze
Suppose we are here:
```
A
```

We choose:
```
A → B
```

Now B is effectively a smaller version of the same problem:
```
"Can I get from B to the exit?"
```

So:
```
FIND_EXIT(A)
```

becomes:
```
FIND_EXIT(B)
```

which becomes:
```
FIND_EXIT(C)
```

That is exactly what recursion is good at:

    Solve the same kind of problem on a smaller/new part of the problem.

And when a path fails, the function returns to the previous invocation.

The stack naturally remembers where each previous invocation was.

That is why the chapter calls the maze a good recursion example.

### The breadcrumb is extremely important
This teaches another deep idea:

#### Recursion alone does not prevent infinite loops.

Suppose:
```
A → B
B → A
```

Then:
```
FIND_EXIT(A)
    ↓
FIND_EXIT(B)
    ↓
FIND_EXIT(A)
    ↓
FIND_EXIT(B)
    ↓
...
```

The solution is:
```
mark visited
```

So:
```
recursive search
+
visited state
```

allows the maze algorithm to terminate correctly.

This is the beginning of what you'll later recognize as graph traversal.

### Maze representation in memory
This is another nice connection to your earlier bit manipulation learning.

Each maze cell is stored in one word.

The book uses bits to encode doors:
```
bit 4 → exit
bit 3 → north
bit 2 → east
bit 1 → south
bit 0 → west
```

And another bit is used as a breadcrumb.

So a cell can conceptually look like:
```
bit15 ... bit4 bit3 bit2 bit1 bit0
       breadcrumb exit north east south west
```

That's a very systems-oriented data representation.

The maze is therefore not magic.

It's just:
```
bits
 ↓
meaning
 ↓
memory
 ↓
algorithm
```

## Queue
Now we move from:
```
LIFO
```

to:
## FIFO
First in, first out

Think about a normal line at a ticket counter
```
A -> B -> C -> D
```
A arrived first.

Therefore A gets served first

So:
```
A removed
then B
then C
Then D
```

that's queue

### Queue needs two ends
A stack needs one important end:
```
TOP
```

A queue needs two:
```
FRONT                  REAR
  ↓                      ↓
[A] [B] [C] [D]
```

The book uses:
```
R3 = FRONT
R4 = REAR
```

Why?

Because:
```
remove from FRONT
insert at REAR
```

That's the definition of the queue behavior.

### Queue implemented in memory
The example reserves:
```
x8000
x8001
x8002
x8003
x8004
x8005
```

Now imagine:
```
45
17
23
74
10
```

The first two may have already been removed.

The queue's logical data might be:
```
23 → 74 → 10
```

while memory still contains:
```
x8000 = 45
x8001 = 17
x8002 = 23
x8003 = 74
x8004 = 10
```

Again:
```
physical contents
        ≠
logical queue contents
```

Exactly like the stack.

### Removing from a queue
Because FRONT points just before the first logical item, removal does:
```
ADD R3,R3,#1
LDR R0,R3,#0
```

Meaning:
```
move FRONT
    ↓
read new front item
```

So:
```
FRONT
  ↓
23  74  10
```

after removal of 23:
```
FRONT
  ↓
74  10
```

### Inserting into the queue
To insert at the rear:
```
ADD R4,R4,#1
STR R0,R4,#0
```

So:
```
move REAR forward
      ↓
store item there
```

### The queue's interesting problem: wasted space
Suppose memory is:
```
x8000
x8001
x8002
x8003
x8004
x8005
```

Initially we put:
```
A B C D E
```

then remove:
```
A
B
```

Now:
```
x8000 → empty
x8001 → empty
x8002 → C
x8003 → D
x8004 → E
x8005 → empty
```
There is space at the beginning.

But REAR may already be at x8004.

So if we only move forward, we'd incorrectly think:

    "I've reached the end."

But x8000 and x8001 are available again.

We need:

### Wrap-around

### Wrap-around
When REAR reaches:
```
x8005
```

and needs another position, it goes back to:
```
x8000
```

Conceptually:
```
x8000 → x8001 → x8002 → x8003 → x8004 → x8005
   ↑                                           |
   |___________________________________________|
```

This creates a circular queue.

That is a very important data-structure pattern.

### Why not use all n slots?
Here comes a subtle design problem.

Suppose there are six allocated locations.

Imagine:
```
FRONT = x8002
REAR  = x8002
```

What does that mean?

Could mean:
```
empty
```

But after wrapping around, it could also mean:
```
full
```

That's ambiguous.

The book solves this by deliberately leaving one slot unused.

For an allocated capacity of:
```
n
```

the queue stores:
```
n - 1
```

elements.

So with:
```
6 memory locations
```

maximum logical elements:
```
5
```


### Empty vs full
With the unused-slot design:
```
Empty
FRONT == REAR
Full
```

After advancing REAR, it would collide with FRONT.

That removes the ambiguity.

So the one unused location is not a mistake.

It is part of the design.

### Queue underflow

Same idea as stack.

If:
```
queue empty
```

and you try:
```
REMOVE
```

you get:
```
UNDERFLOW
```

### Queue overflow

If:
```
queue full
```

and you try:
```
INSERT
```

you get:
```
OVERFLOW
```

The chapter's queue subroutine reports success/failure using:
```
R5 = 0 → success
R5 = 1 → failure
```

just like the stack routines.

54. Complete queue mental model
```
Remember:

                  QUEUE

        remove                 insert
           ↓                      ↓
        FRONT                  REAR

[A] [B] [C] [D] [E] [ ]

        FIFO
```

And with wrap-around:
```
        ┌───────────────────────────────┐
        ↓                               |
x8000 → x8001 → x8002 → x8003 → x8004 → x8005
  ↑                                         |
  └─────────────────────────────────────────┘
```