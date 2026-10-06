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

