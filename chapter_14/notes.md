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