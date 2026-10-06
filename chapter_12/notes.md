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