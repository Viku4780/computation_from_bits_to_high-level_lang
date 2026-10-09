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