    A function is an abstraction that lets us use a piece of computation without constantly thinking about how that computation is implemented


## Why do we even need functions?
Imagine you have this program:

```
1. Read a number
2. Calculate its square
3. Print the square

3. Read another number
5. Calculate its square
6. Print the square

7. Read another number
8. Calculate its square
9. Print the square
```

The operation
```
calculate square
```
keeps appearing.

you could write:
```
x * x
```

every time.

BUt imagine the opertion becomes complicated.

Suppose "calculate square" eventually becomes:
```
check input
convert type
perform calculation
handle special cases
check overflow
...
```

Now imagine writing all of that every time you need a square.

That's bad software design.

Instead, we create a reusable component:
```
Squared(x)
```

and then simply write:
```
Sqaured(a)
Sqaured(b)
Sqaured(c)
```

The details are hidden inside Squared.

    A function is a named, reusable unit of computation that can recieve values, perform operations, and optionally return a value.


### Functions lets us create our own building blocks
C already gives you operations:
```
+
-
*
/
%
```

and control structures:
```
if
while
for
switch
```

Functions allow you to create your own higher-level building blocks.

For example:
```
int Squared(int x)
{
    return x * x;
}
```
Now you have effectively created a new operation:
```
Squared(x)
```

Then:
```
Squared(5)
```

Means:
```
25
```

This is why the textbook describes functions as allowing programmers to create new primitive builing blocks.

### A C program is essentially a collection of functions
Consider:
```
#include <stdio.h>

void SayHello(void)
{
    printf("Hello\n");
}

int main(void)
{
    SayHello();
    return 0;
}
```

There are two functions:
```
SayHello
main
```

Execution begins in:
```
main()
```

Then:
```
main()
   │
   └── calls SayHello()
             │
             ▼
          prints Hello
             │
             ▼
        returns to main
```

The book emphasizes that every C statment belongs to a function, execution begins in main, and functions can call other functions, which can themselves call other functions. Eventually control returns to main, and when main finishes, the program finishes.


## Four important words
These are extremely important:

1. Declaration
2. Definition
3. Call
4. Return

### Function declaration
Suppose we want a function:
```
int Add(int a, int b);
```

This is a function declaration.

it tells the compiler:

    There exists a function called Add.

And:
```
name     == Add
return   == int
inputs   == int, int
```

So:
```
int Add(int a, int b);
```

means:
```
Add
 ├── accepts int
 ├── accepts int
 └── returns int
```

The declaration does not provide the implementation.

its basically saying:

    Compiler, trust me for now. A function with this interface exists.


### Function prototype
In ordinary C terminology, this:
```
int Add(int a, int b);
```

is a function prototype.

A prototype tells the compiler:
```
function name
return type
number/type of parameters
```

You can write:
```
int Add(int, int);
```

too.

The parameter names aren't required in a prototype.

For example:
```
int Add(int, int);
```

and:
```
int Add(int a, int b);
```
both communicate the parameter types.

The names:
```
a
b
```

are useful for readability, but the prototype primarily establishes the function's interface.


### Function definition
Now we actually implement the function:
```
int Add(int a, int b)
{
    return a + b;
}
```
This is the definition.

It tells the compiler exactly what the function does.

Think:
```
DECLARATION
    ↓
"What exists?"

DEFINITION
    ↓
"How does it work?"
```

### Function call
Now:
```
int result = Add(10, 20);
```
This is a function call.

We're asking the computer:

    Execute Add using 10 and 20.

So:
```
Add(10, 20)
```
causes execution to move into:
```
int Add(int a, int b)
{
    return a + b;
}
```

### Arguments vs parameters
Consider:
```
int Add(int a, int b)
{
    return a + b;
}
```

Here:
```
a
b
```
are called parameters.

They are the variables that receive the input.

Now:
```
Add(10, 20);
```

Here:
```
10
20
```
are arguments.

Remember:

FUNCTION DEFINITION:
```
int Add(int a, int b)
            ↑     ↑
        parameters

```

FUNCTION CALL:
```
Add(10, 20)
    ↑   ↑
  arguments
```

Simple rule:

    Parameters belong to the function definition. Arguments are the actual values supplied during the call.

### Complete simple example
```
#include <stdio.h>

int Add(int a, int b);

int main(void)
{
    int result;

    result = Add(10, 20);

    printf("%d\n", result);

    return 0;
}

int Add(int a, int b)
{
    return a + b;
}
```

#### Step 1 — program begins
Execution begins:
```
main()
```

We have:
```
int result;
```
So memory is allocated for:
```
result
```

#### Step 2 — function call
Then:
```
result = Add(10, 20);
```

The CPU must now execute another function.

Conceptually:
```
main
 │
 │ calls Add(10,20)
 ▼
Add
```

### What happens to 10 and 20?
The function has:
```
int Add(int a, int b)
```

So:
```
a = 10
b = 20
```

Conceptually:
```
Add's private execution area

a → 10
b → 20
```

Then:
```
return a + b;
```

becomes:
```
return 30
```

### Where does 30 go?
It goes back to the caller:
```
result = Add(10, 20);
```

So:
```
Add
 │
 │ returns 30
 ▼
main
```

and:
```
result = 30;
```

Then:
```
printf("%d\n", result);
```

prints:
```
30
```

### Functions can have no parameters
Example:
```
void PrintHello(void)
{
    printf("Hello\n");
}
```
Call:
```
PrintHello();
```

Here:
```
parameters = none
return value = none
```

But notice something important

There are two voids:
```
void Printello(void)
```

First void:
```
function returns nothing
```

Second void:
```
function accepts no arguments
```
This is worth remembering.

```
void PrintHello(void)
│              │
│              └── accepts no parameters
│
└── returns no value
```

### Function returning a value
Example:
```
int Square(int x)
{
    return x * x;
}
```
Call:
```
int result = Square(5);
```

execution:
```
Square(5)
   │
   ▼
x = 5
   │
   ▼
x * x
   │
   ▼
25
   │
   ▼
return 25
```

Then:
```
result = 25
```

### return has two jobs
Consider:
```
return x * x;
```
it does two things conceptually:

#### Job 1
finish execution of the current function.

#### job 2
provide a value to the caller.

For example:
```
return 25;
```

means:
```
function stops
     +
25 is given back
```

### Local variables belong to a function invocation
Consider:
```
int Square(int x)
{
    int result;

    result = x * x;

    return result;
}
```

result is a local variable.

it belongs to that particular execution of Square.

if we call:
```
Square(5)
```

one execution gets its own:
```
x
result
```

if we later call:
```
Square(10)
```

that execution gets its own local execution state.

this idea becomes extremely important when we reach:

- recursion
- stack frames
- pointers
- memory
- multitasking


## Why can't functions simply use fixed memory?
Suppose:
```
int Square(int x)
{
    int result;
    ...
}
```
Where does x live?

Where does result live?

One possibility would be:
```
Square.x      → fixed memory
Square.result → fixed memory
```

But what happens if Square() calls itself?

For example:
```
Square(...)
   ↓
Square(...)
      ↓
Square(...)
```

Now there are multiple active executions of the same function.

If all of them use the same memory:
```
Square.x
Square.result
```
they would overwrite each other.

So we need a different solution.


### The runtime stack
The solution is the runtime stack.

You already learned that the stack stores things such as function execution state.

Now you get to see why.

Imagine:
```
main()
{
    ...
    Square(5);
}
```

When main is running:
```
STACK

┌──────────────┐
│ main frame   │
└──────────────┘
```

Then main calls Square.

A new frame is placed on the stack:
```
STACK

┌──────────────┐
│ Square frame │
├──────────────┤
│ main frame   │
└──────────────┘
```

When Square finishes:
```
STACK

┌──────────────┐
│ main frame   │
└──────────────┘
```
The Square frame disappears.

This is exactly why the runtime stack is so useful.

The textbook explains this as the mechanism that allows each function invocation to have its own local storage, including recursive calls.


### Stack frame
A function's runtime information is grouped into what we call a:

    stack frame

or:

    activation record

Think of it as the function's temporary workspace.

A simplified frame may contain things such as:
```
┌─────────────────────────┐
│ arguments / parameters  │
├─────────────────────────┤
│ return value            │
├─────────────────────────┤
│ return address          │
├─────────────────────────┤
│ saved frame pointer     │
├─────────────────────────┤
│ local variables         │
├─────────────────────────┤
│ other bookkeeping       │
└─────────────────────────┘
```

The exact layout depends on the architecture and calling convention. The textbook uses a specific LC-3 calling convention to demonstrate these ideas.

### Why do we need a return address?
Imagine:
```
main()
{
    int x;

    x = Square(5);

    printf("%d", x);
}
```

When main calls Square, the CPU must eventually return to:
```
x = Square(5);
```

But how does it know where to return?

It needs a:

    return address

Conceptually:
```
main
   │
   │ call Square
   │
   ├──────────────► Square
   │                 │
   │                 │ calculate
   │                 │
   ◄─────────────────┘
   │
   │ continue here
   ▼
printf(...)
```

The return address tells the processor:

    "When this function finishes, resume execution from this location."


### This connects directly to assembly
You already learned LC-3 subroutines.

At the assembly level, a subroutine call involves something like:
```
JSR
```

and returning:
```
RET
```

The textbook explicitly says:

    C functions are the high-level equivalent of subroutines at the LC-3 machine level.

So:
```
C

Add(10, 20)
```

is conceptually implemented using mechanisms like:
```
save return location
transfer control
execute function
produce return value
restore previous state
return
```


## The four fundamental phases of a function call

#### Phase 1 - Pass arguments
Caller gives the callee the required values.

```
main → Add
      10
      20
```

#### Phase 2 - Transfer control
execution moves from caller to callee
```
main
 ↓
Add
```

#### Phase 3 - Execute
The function performs its work.
```
10 + 20
```

#### Phase 4 - Return
Control comes back, potentially with a result

```
Add
 ↓
30
 ↓
main
```

### Caller and callee
Two important words:
```
caller
callee
```

Suppose:
```
main()
{
    Add(10, 20);
}
```

Then:
```
main = caller
Add  = callee
```

Why?

Because:

    main calls Add.

So:
```
caller
   │
   ▼
callee
```

This terminology becomes very important when we discuss stack frames and calling conventions.


### Calling convention
The computer needs rules for answering questions like:

- Where are arguments placed?
- Where is the return value placed?
- Where are local variables stored?
- Where is the return address stored?
- Which registers must be preserved?
- How does the caller know where the callee's data is?
- How does the callee return safely?

These rules form a:

    calling convention


### Stack pointer vs frame pointer
You have already encountered these concepts in your memory studies, but Chapter 14 gives them a specific function-call meaning.

The textbook's LC-3 convention uses:
```
R5 = frame pointer
R6 = stack pointer
R7 = return address
```

Conceptually:

#### Stack pointer

Points to the current top of the stack.
```
SP
 ↓
┌──────────────┐
│ newest data  │
└──────────────┘
```

#### Frame pointer
Provides a stable reference point inside the current function's frame.
```
FP
 ↓
┌─────────────────┐
│ function frame  │
│                 │
│ locals          │
│ arguments       │
└─────────────────┘
```

Why do we need both?

Because the stack pointer moves as things are pushed and popped.

The frame pointer gives the current function a more stable reference point.

The textbook specifically describes R5 as the frame pointer and R6 as the stack pointer.


## Let's visualize a function call at the machine level
Suppose:
```
int result = Add(10, 20);
```

Conceptually:
#### Before call
```
STACK

┌────────────────┐
│ main frame     │
└────────────────┘
```

#### Caller prepares arguments
```
STACK

┌────────────────┐
│ 20             │
├────────────────┤
│ 10             │
├────────────────┤
│ main frame     │
└────────────────┘
```

#### Control transfers
```
main
 ↓
Add
```

#### Add creates its frame
Conceptually:
```
STACK

┌────────────────┐
│ Add locals     │
├────────────────┤
│ saved state    │
├────────────────┤
│ return value   │
├────────────────┤
│ 20             │
├────────────────┤
│ 10             │
├────────────────┤
│ main frame     │
└────────────────┘
```

The exact LC-3 ordering is more specific, but this is the correct conceptual picture.


### Then Add executes
```
return a + b;
```

Conceptually:
```
a = 10
b = 20

a + b
   ↓
  30
```

The return value is prepared.

Then the callee restores the caller's environment and returns.

The caller recieves:
```
30
```

and stores it:
```
result = 30;
```

### Why is the stack such a beautiful solution?
Because function calls naturally behave like a stack.

Suppose:
```
main()
{
    A();
}

A()
{
    B();
}

B()
{
    C();
}

C()
{
}
```

Execution:
```
main
 ↓
 A
 ↓
 B
 ↓
 C
```

Stack:
```
C frame
────────
B frame
────────
A frame
────────
main frame
```

Then C returns:
```
B frame
────────
A frame
────────
main frame
```

Then B returns:
```
A frame
────────
main frame
```

Then A returns:
```
main frame
```

This is exactly:
```
Last In
First Out
```

which is the definition of a stack.

The textbook illustrates the runtime stack growing and shrinking as functions are called and return.


## C uses pass-by-value
Consider:
```
void Change(int x)
{
    x = 100;
}
```

And:
```
int main(void)
{
    int a = 10;

    Change(a);

    printf("%d\n", a);
}
```

What do you think prints?
```
10
```

Why?

Because:
```
Change(a);
```

passes the value of a.

Conceptually:
```
main:
a = 10

       copy
        ↓

Change:
x = 10
```

NOw: 
```
x = 100;
```

changes:
```
Change's x
```
not:
```
main's a
```

So:
```
main's a = 10
```

remains unchanged.


### Think in terms of copies
This is the simplest mental model:
```
int a = 10;

Change(a);
```

means approximately:
```
a
│
│ value = 10
│
└──────copy──────► x
                   │
                   │ value = 10
                   ▼
                 modify x
```

So:
```
a ≠ x
```

They are separate objects.

This is why this program:
```
void Swap(int x, int y)
{
    int temp;

    temp = x;
    x = y;
    y = temp;
}
```

does not swap the caller's variables.

### Parameters are local to the function
Consider:
```
int Add(int a, int b)
{
    return a + b;
}
```

a and b belong to the function invocation.

You cannot just use them from main:
```
int main(void)
{
    printf("%d", a);  // ERROR
}
```

because a doesn't exist in main's scope.


## Function interface
Suppose we have:
```
int CalculateTax(int income);
```

The caller needs to know:
```
function name
input type
return type
```

The caller does not necessarily need to know how tax is calculated.

So the function provides an interface.

```
                INTERFACE
                   │
        ┌──────────┴──────────┐
        │                     │
    input type            output type
        │                     │
       int                   int
        │                     │
        └──── CalculateTax ───┘
```
implementation:
```
int CalculateTax(int income)
{
    // complicated implementation
}
```

The caller only cares about:
```
CalculateTax(income)
```

### Why declaration before definition?
Consider:
```
int main(void)
{
    int result = Add(2, 3);
}

int Add(int a, int b)
{
    return a + b;
}
```

Depending on the C language version/compiler rules, the compiler needs to know about Add before the call.

So we typically write:
```
int Add(int a, int b);
```

before main.

Then:
```
int main(void)
{
    int result = Add(2, 3);
}
```

and later:
```
int Add(int a, int b)
{
    return a + b;
}
```

So:
```
declaration
     ↓
compiler learns interface
     ↓
call
     ↓
definition supplies implementation
```

### Function decomposition
Suppose you need to write:
```
Student management system
```

Don't write one enormous main.

Instead:
```
main()
 │
 ├── ReadStudent()
 │
 ├── CalculateAverage()
 │
 ├── FindHighestScore()
 │
 ├── PrintStudent()
 │
 └── SaveStudent()
```

Now each function has a responsibility.

This is called decomposition.

You take:
```
large problem
```

and break it into:
```
smaller problems
```
Then solve each independently.