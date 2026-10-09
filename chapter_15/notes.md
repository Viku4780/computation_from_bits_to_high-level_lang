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


### Semantic errors
Suppose:
```
int x = 10;
int y = 20;

int result = x - y;
```

The program:

- compiles,
- executes,
- produces a value.

Nothing is syntactically wrong.

But suppose we wanted:
```
x + y
```
and accidentally wrote:
```
x - y
```
The program is syntactically valid.

But it does the wrong thing.

That's a semantic error.

### Another semantic example
Suppose the program is supposed to print:
```
1 × 7 = 7
2 × 7 = 14
...
10 × 7 = 70
```

We write:
```
for (i = 0; i <= 10; i++)
{
    j = i * 7;
    printf("%d x 7 = %d\n", i, j);
}
```
This compiles.

But output starts:
```
0 x 7 = 0
1 x 7 = 7
...
```
Maybe we wanted to start at 1.

The syntax is valid.

The program executes.

But it doesn't produce the intended result.


### Algorithmic errors
An algorithm error means:

    Our approach to solving the problem is wrong.

This is different from simply mistyping something.

Suppose the problem is:

    Find the average of 100 numbers.

You decide:
```
Add all numbers
Divide by 2
```

you implement that perfectly:
```
average = total /2;
```
No syntax error

no typo

the code faithfully implements your algorithm.

But your algorithm is wrong.

Correct:
```
average = total / 100
```

So:
```
wrong algorithm
      ↓
correct implementation
      ↓
wrong result
```

This is an algorithm error.

```
syntax error
    ↓
usually compiler catches it

semantic error
    ↓
testing reveals it

algorithmic error
    ↓
requires understanding the problem and algorithm
```


### Specification errors
Suppose a client says:

    Build a system that calculates employee bonuses

You interpret the requirement as:
```
Bonus = 10% of salary
```

you build the system perfectly.

Later the client says:

    No. Bonuses are 10% only when performance is above 80%; otherwise 5%

your code may be:
- syntactically correct,
- semantically correct,
- algorithmically correct,

accoding to your interpretation

But the requirements itself was misunderstood

that's a:

    Specification error

```
                SPECIFICATION
                "What is required?"
                       ↓
                  ALGORITHM
                "How will we solve it?"
                       ↓
                  PROGRAM
                "How do we express it?"
                       ↓
                  MACHINE
                "How does it execute?"
```