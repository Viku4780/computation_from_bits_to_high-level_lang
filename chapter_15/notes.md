Chapter 15 teaches us:

    How to determine whether the program is actually correct, find where it is wrong, and systematically fix it.

#### How do we reason about a program when reality does not match our expectation?

## What is bug?
Suppose we write:
```
int result = 10 + 20;
```

We expect:
```
30
```

But program produces:
```
40
```
Something is wrong

We call that a:

    bug

A bug is essentially a defect in a system that causes it to behave differently from what is required or intended.

BUt  there is something subtle here.

A program can:

- compile successfully.
- start successfully,
- execute successfully,

and still be wrong


### Three different questions
when we write a program, we should ask three different questions.

#### Question 1
 
    Is the code valid C?

This is about syntax.

#### Question 2

    Does the program implement the correct algorithm?

This is about algorithmic correctness.

#### Question 3

    does the program satisfy what the user actually wanted?

This is about the specification.

These are different.

For example:
```
average = total / 2;
```
might be perfectly valid C.

The compiler may accept it.

But suppose the number of students is actually stored in :
```
count
```

then the correct calculation should be:
```
average = total / count;
```

The first program can be syntactically perfect and still wrong.


### Syntactic errors
Suppose:
```
#include <stdio.h>

int main(void)
{
    int x

    printf("%d\n", x);

    return 0;
}
```

Look carefully:
```
int x
```
is missing:
```
;
```
Correct:
```
int x;
```

This is syntax error.

why?

Because the code doesn't conform to the grammer of C.

### Examples of syntax errors

#### Missing semicolon
```
int x
```

#### Missing brace
```
if (x > 5)
{ 
    printf("yes");
```

#### Wrong syntax
```
if x > 5
```

instead of:
```
if (x > 5)
```

#### Invalid declaration
```
int;
```

#### Misspelled keyword
```
whlie(x < 10)
```

instead of:
```
while(x < 10)
```

These are generally caught before the program can execute.

### But compiler errors aren't always as simple as they look
Suppose:
```
int x
int y;
```

The compiler might complain about:
```
expected ';'
```

that's useful

but sometimes the compiler points at a location after the real mistake.

For example:
```
int x
printf("hello");
```

The compiler might identify printf as the problematic location.

BUt the actual problem is :
```
int x

missing ;
```

Why?

Because the compiler is parsing a sequence of tokens and only realizes something has gone wrong the next token doesn't make sense.4

So:
 
    The reported line is not always the root cause.