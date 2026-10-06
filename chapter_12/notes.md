    Variables hold data. Operations manipulate that data.


    Where do those C variables actually live in memory, and what does the compiler generate to operate on them?


```
                  C PROGRAM
                      │
          ┌───────────┴───────────┐
          │                       │
       VARIABLES              OPERATORS
          │                       │
      hold values          manipulate values
          │                       │
          └───────────┬───────────┘
                      │
                  EXPRESSIONS
                      │
                  STATEMENTS
                      │
                      ▼
                 C COMPILER
                      │
          ┌───────────┴───────────┐
          │                       │
      SYMBOL TABLE           MEMORY MAP
          │                       │
          └───────────┬───────────┘
                      ▼
                 LC-3 CODE
                      │
                      ▼
              MEMORY + REGISTERS
```

### A Value
A value is simply some piece of information that a program works with.

examples:
```
5
42
-17
'A'
3.14
true
```

A program may use these values as:
```
loop counters
user input
prices
temperatures
characters
calculation results
flags
```

At the machine level, all of these ulitmately become bit patterns.

But C gives those bit patterns names and types.

That is where variables come in.


## What is a variable?
Consider:
```
int score;
```

You can think of this as saying:

    "Create a named memory object called score that holds an integer."

The book describes variables as the most basic kind of memory object. A variable gives the programmer a symbolic name instead of forcing the programmer to remember a raw memory location.

Without variables, you might have to think:
```
memory location x5023 contains the score
```

With C:
```
score
```

### The important hidden information inside a variable declaration
When the compiler sees:
```
int score;
```
it needs several pieces of information.

At minimum:
```
name -> score
type -> int
scope -> where score is visible
location -> where storage will be allocated
```

The source explains that the declaration gives the compiler the identifier and type explicitly, while scope is determined from where the declaration occurs.

This becomes the basis of the compiler's symbol table.

### A variable is not the same thing as its name

When you write:
```
int score;
```

score is the name we use in source code.

The actual object exists somewhere in the machine's storage system.

Conceptually:
```
C source:

score
  │
  ▼
compiler's symbol information
  │
  ▼
some storage location
  │
  ▼
binary value
```

So:
```
score
```
is your programmer-friendly handle for some underlying storage.

### Types: why do we need them?
Suppose memory contains:
```
01100110
```

What is that?

It could represent:
```
102
```
as an integer.

Or:
```
'f'
```
as ASCII.

The bits themselves do not carry a magical sticker saying:
```
"I am an integer!"
```
The type/context tells the compiler how the bits should be interpreted and what operations should be performed on them.


## The four basic C types in this chapter
```
int
char
float / double
_Bool / bool
```

### int
Example:
```
int numberOfSeconds;
```

This means the variable contains an integer.

On the LC-3 used by the book:
```
int = 16 bits
```

and because LC-3 integers are represented using 2's complement:
```
minimum = -32768
maximum = +32767
```

The book points out that the size and range of int depend on the underlying hardware. On a typical 32-bit-int system, the range is much larger.

This connects directly to what you learned about fixed-width integers.

### Why does the ISA affect int?
Because C needs to eventually represent the integer using the machine's available storage.

The book's simplified model is:
```
LC-3
  ↓
16-bit word
  ↓
int uses one LC-3 memory location
```

On another architecture:
```
different architecture
  ↓
different representation/size
```

This is one reason the exact size of C's ordinary types should never be blindly assumed to be identical everywhere.

### char
Example:
```
char key = 'Q';
```

A char represents a character.

Notice:
```
'Q'
```

uses single quotes.

That means:
```
character literal
```

The character has an underlying numeric representation, such as its ASCII code.

So conceptually:
```
'Q'
 ↓
ASCII code
 ↓
binary value
```

### '4' is not the same as 4
Consider:
```
char a = '4';
int b = 4;
```

These represent different things.
```
'4'
 ↓
character code for the character 4

4
 ↓
integer value four
```

This is exactly the same kind of representation-vs-meaning distinction you saw in Chapter 10.

The source explicitly uses examples such as:
```
char number = '4';
```
to illustrate character literals.

### How much memory does char use?
The textbook makes a deliberate simplification for the LC-3:
```
char = 16 bits
```

even though ASCII itself only needs 8 bits.

Why?

Because one LC-3 memory location is 16 bits, and the book wants to keep its examples straightforward.

So don't confuse:
```
"How many bits does ASCII require?"
```

with:
```
"How many bits does this book allocate for a C char in its LC-3 model?"
```

They are different questions.

### float and double
These represent floating-point values.

Examples:
```
double temperature;
double cost;
double average;
```

A floating-point representation conceptually contains components for:
```
sign
fraction / mantissa
exponent
```

The exact encoding depends on the representation standard and implementation.

### Floating-point literals
The chapter gives examples such as:
```
double twoPointOne = 2.1;
double twoHundredTen = 2.1E2;
double twoHundred = 2E2;
double twoTenths = 2E-1;
double minusTwoTenths = -2E-1;
```

Let's understand the E notation.
```
2.1E2
```
means:
```
2.1 × 10²
```
so:
```
210
```
And:
```
2E-1
```
means:
```
2 × 10⁻¹
= 0.2
```
Therefore:
```
E2   → multiply by 100
E-1  → multiply by 0.1
```
The exponent itself is an integer.

### float vs double
The chapter says:
```
float
 ↓
single precision

double
 ↓
double precision
```

Generally, double provides at least as much precision/range as float.

The book notes that the exact sizes depend on the compiler and ISA, while commonly:
```
float  = 32 bits
double = 64 bits
```
under common IEEE 754 implementations.

### Boolean values
A Boolean answers a yes/no question.

Examples:
```
true
false
```

In C, the basic type is:
```
_Bool
```

The convenient:
```
bool
```

form is made available through:
```
#include <stdbool.h>
```

with:
```
true  → 1
false → 0
```

The chapter gives examples:
```
_Bool flag = 1;
bool test = false;
```

## Scope — where is a variable visible?
Suppose:
```
int main(void)
{
    int score = 10;

    printf("%d", score);
}
```

score exists within the block belonging to main.

But if you have another block:
```
{
    int x;
}
```
then x belongs to that block.

The basic idea of scope is:

    Which part of the program is allowed to refer to this variable by name?

### What is a block?
A block is code enclosed in:
```
{
    ...
}
```

For example:
```
int main(void)
{
    int x;

    {
        int y;
    }
}
```

There are two nested blocks.

Conceptually:
```
main block
│
├── x
│
└── inner block
    └── y
```

### Local variables
A variable declared inside a block is local to that block.

the compiler treats the variable as belonging to a particular scope.

### Nested scope
Now look at this:
```
int globalVar = 2;

int main(void)
{
    int localVar = 3;

    {
        int localVar = 4;

        printf("%d\n", localVar);
    }

    printf("%d\n", localVar);
}
```

Inside the inner block:
```
localVar
```

means the inner variable:
```
localVar = 4;
```

Outside that inner block:
```
localVar
```

means the outer one:
```
localVar = 3
```

So the same identifier can name different variables in different scopes.

### Shadowing
This behavior is commonly called shadowing.

Visualize:
```
outer block

localVar → 3
     │
     └─────────────┐
                   ▼
             inner block
             localVar → 4
```

inside the inner block, the nearer declaration wins.

When the inner block ends, the outer variable becomes visible again.

### Global variables
Now:
```
int globalVar = 2;

int main(void)
{
    ...
}
```

Because it is declared outside the blocks/functions in this example, it is a global variable in the textbook's simplified treatment.

global variables can be accessed by many parts of the program.

### Why globals can become dangerous
Imagine:
```
function A changes global x
        ↓
function B reads x
        ↓
function C changes x
        ↓
function D depends on x
```

Now understanding one function requires knowing what other parts of the program might have done to the global.

This creates hidden coupling.

The textbook warns that global variables can make large programs harder to debug, maintain, extend, and modify, and therefore deliberately minimizes their use.

### Local initialization vs global initialization
Consider:
```
int x;
```

What value does x have?

It depends on where x is declared.

#### Local variable

A local variable without an initializer has an undefined/indeterminate value in the book's terminology, often informally called a garbage value.

Example:
```
int main(void)
{
    int x;
}
```
Do not assume:
```
x = 0
```

#### Global variable
A global variable without an explicit initializer is initialized to:
```
0
```

So:
```
int x;
```

at global scope gives zero initialization.

therefore initialize local variables


## Now we reach operators
Variables give you data.

Operators give you ways to manipulate that data.

for example:
```
score = score + 3;
```

contains:
```
score       → variable
=           → assignment operator
+           → addition operator
3           → literal
score + 3   → expression
```


### Expression vs statement
Consider:
```
x + y
```

This is an expression.

it produces a value.

Now:
```
z = x + y;
```

is a statement.

it represents a complete unit of work.

Think:
```
expression
 ↓
produces a value

statement
 ↓
performs an action
```

### Compound statements / blocks
Multiple declarations and statements can be grouped:
```
{
    a = b + c;
    i = p * r * t;
}
```

This is a compound statement, also called a block.

So {} are not merely visual decoration.

They also establish program structure and scope.


### The assignment operator =
This is probably the first operator beginners misunderstand.

Consider:
```
a = b + c;
```

C means:

    Evaluate b + c, then store that resulting value into a.

So:
```
b + c
  ↓
calculate
  ↓
value
  ↓
store in a
```

### Assignment is not mathematical equality
This is extremely important.

In mathematics:
```
a = b + c
```

means:

    a and b+c represent equal values.

In C:
```
a = b + c;
```

means:

    calculate the right side and make a contain that result.

So:
```
x = x + 4;
```

makes perfect sense in C.

It means:
```
old x
 ↓
add 4
 ↓
new x
```

Mathematically, you might object:
```
x = x + 4
```

cannot be an equality.

But it is not being used as an equality assertion.

It is an assignment operation.


### How assignment becomes LC-3
Suppose:
```
x = x + 4;
```

and R5 contains the address of x.

The textbook's compiler generates:
```
LDR R0, R5, #0
ADD R0, R0, #4
STR R0, R5, #0
```

Let's decode it:
```
LDR
 ↓
get x from memory

ADD
 ↓
x + 4

STR
 ↓
put result back into x
```

### The C compiler is doing the bookkeeping
You wrote:
```
x = x + 4;
```

You didn't write:
```
LDR R0, ...
ADD R0, ...
STR R0, ...
```
The compiler figures that out.

That is the essence of compilation.

### Arithemetic operators
```
+ , -, *, /, %
```

### Precedence vs associativity
Don't mix them up.

#### Precedence
Answers:

    Which operator group gets priority?

Example:
```
* before +
```

#### Associativity
Answers:

    When operators have the same precedence, which direction do we evaluate?

Example:
```
+ and -
```
are evaluated left-to-right.


#### Bitwise
```
& | ^ ~ << >>
```

means:

    Work with the bits themselves.

#### Logical
```
&& || !
```

means:

    Work with truth/falsity.


### Combining relational + logical operators
Suppose you want to test:

    Is x between 10 and 20 inclusive?

You can write:
```
(10 <= x) && (x <= 20)
```

Why?

First condition:
```
10 <= x
```

Second:
```
x <= 20
```
Both need to be true.

That's exactly what && represents.

### Testing whether a character is a letter
The chapter gives the idea:
```
(('a' <= c) && (c <= 'z')) ||
(('A' <= c) && (c <= 'Z'))
```

This means:
```
is c a lowercase letter?
          OR
is c an uppercase letter?
```

Again:
```
relational operators
       ↓
produce true/false
       ↓
logical operators combine them
```

### Increment and decrement
C has:
```
++
--
```

These exist because incrementing and decrementing are so common.
```
x++;
```

means essentially:
```
x = x + 1;
```

and:
```
x--;
```

means essentially:
```
x = x - 1;
```

### Post-increment vs pre-increment

Consider:
```
x = 4;
y = x++;
```

What happens?

The old value is used by the expression first.

So:
```
y = 4
x = 5
```

Think:
```
use x
 ↓
then increment x
```

### Pre-increment
Now:
```
x = 4;
y = ++x;
```

The increment happens first.

So:
```
x = 5
y = 5
```

Think:
```
increment x
 ↓
then use x
```

## Now lets solve a real problem
Suppose:
```
amount = number of bytes
rate = bytes per second
```

We want:
```
hours 
minutes
seconds
```
for the transfer.

This is a perfect example because it demonstrate how to go from:
```
problem
 ↓
algorithm
 ↓
data representation
 ↓
operators
 ↓
C code
```

### Step 1 - understand the problem
We need threee broad phases:
```
get input
calculate
output
```

This is first refinement

### Step 2 -  refine calculation
Calculation becomes:
```
calculate total time in seconds
convert total seconds to
hour, minutes, seconds
```

Still not quite enough.

### Step 3 -  refine conversion
Break it down further:
```
total seconds
      ↓
calculate hours
      ↓
remaining seconds
      ↓
calculate minutes
      ↓
remaining seconds
      ↓
final seconds
```

now we are close enough write C code.

This is the stepwise refinement methodology

### Choosing the data type
We need hours, minutes, and seconds

do we need floating point?

probably not.

for example:
```
10.7 hours
```

is not how we want to represent this result.

We want:
```
10 hours 42 minutes ...
```
Therefore integers are a natural representation

This is an important programming principle:

    Choose a data representation that naturally matches the problem.


### The complete network program
```
#include <stdio.h>

int main(void)
{
    int amount;
    int rate;
    int time;
    int hours;
    int minutes;
    int seconds;

    printf("How many bytes of data to be transferred? ");
    scanf("%d", &amount);

    printf("What is the transfer rate (in bytes/sec)? ");
    scanf("%d", &rate);

    time = amount / rate;

    hours = time / 3600;
    minutes = (time % 3600) / 60;
    seconds = (time % 3600) % 60;

    printf("Time : %dh %dm %ds\n", hours, minutes, seconds);

    return 0;
}
```

## Now the chapter goes underneath C
The book now asks:

    We have variables in C. Where do those variables actually get allocated?

Remember:
```
int amount;
int rate;
int time;
```

These need storage.

The compiler must decide where that storage belongs.


### The symbol table

The compiler maintains a symbol table.

For a variable, the textbook's simplified symbol-table entry contains:

identifier
type
memory location/offset
scope
other information

For example:

amount → int → offset 0 → main
rate   → int → offset -1 → main
time   → int → offset -2 → main
...

### Why use offsets instead of absolute addresses?
Because local variables are associated with a function's stack frame.

Suppose:
```
R5 = base of current stack frame
```

and :
```
amount -> offset 0
rate -> offset -1
time -> offset -2
```

then the compiler can generate:
```
LDR R0, R5, #0
```
for amount.

for time:
```
LDR R0, R5, #-2
```

The actual absolute memory address can change depending on where the stack frame lives.

The relative offset remains useful.

This is very important compiler technique.

### Local variables live in the runtime stack
The chapter states:
```
global variables
    ↓
global data section

local variables
    ↓
runtime stack
```

So:
```
int main(void)
{
    int x;
}
```
means x is associated with main's stack frame in the textbook's LC-3 model.

## What is a stack frame?
A stack frame, also called an activation record, is a contiguous region of the runtime stack associated with one function invocation.

For example:
```
main's stack frame

+------------------+
| amount           |
+------------------+
| rate             |
+------------------+
| time             |
+------------------+
| hours            |
+------------------+
| minutes          |
+------------------+
| seconds          |
+------------------+
```

### R5 and R6
The textbook uses:
```
R5 = frame pointer
R6 = stack pointer
```

#### R5

Points into the current stack frame.

It gives the compiler a stable reference point for local-variable offsets.

#### R6

Points to the top of the runtime stack.

So:
```
R5 → current frame
R6 → top of stack
```

### Why do we need a frame pointer?
Suppose:
```
R5 = 0x8000
```

and:
```
amount → R5 + 0
rate   → R5 - 1
time   → R5 - 2
```

Then the compiler knows:
```
amount = M[0x8000]
rate   = M[0x7FFF]
time   = M[0x7FFE]
```

If another function is called, a new stack frame is created and R5 changes to refer to that new frame.

This is why the compiler can use simple offsets for local variables.

### Why are the offsets negative?
This is because the stack in the LC-3 model grows toward lower addresses.

So conceptually:
```
higher addresses
       ↑

      R5
       │
       ├── offset 0
       ├── offset -1
       ├── offset -2
       ├── offset -3
       ↓
lower addresses
```

That "upside-down" appearance is simply a consequence of the stack growing toward address x0000.