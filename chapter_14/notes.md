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