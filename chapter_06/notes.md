This chapter teaches us:

    How do i take a problem, turn it into a program, and then systematically find what i got wrong?

The book calls this systematic decomposition, also called stepwise refinement, and then applies the same disciplined thinking to debugging.

# Programming
The chapter has two major parts:
```
6.1 Problem Solving
    ↓
    How to construct a program

6.2 Debugging
    ↓
    How to find and fix mistakes
```

the most important idea of the entire chapter is:
```
Problem
   ↓
Understand the problem
   ↓
Algorithm
   ↓
Break algorithm into smaller tasks
   ↓
Break those tasks into even smaller tasks
   ↓
Simple operations
   ↓
LC-3 instructions
```

And when the program is wrong:
```
Program
   ↓
Observe actual behavior
   ↓
Compare with expected behavior
   ↓
Locate the mismatch
   ↓
Identify the instruction causing it
   ↓
Fix it
   ↓
Test again
```


## First understand what an algorithm actually is
Suppose i tell you:

    Make tea.

That is problem statement.

But a computer cannot directly execute make tea.

We need to turn it into precise steps:
```
1. Get water.
2. Heat water.
3. Put tea leaves in container.
4. Pour hot water.
5. Wait.
6. Add milk.
7. Add sugar.
```
This is moving from a problem statement to an algorithm.

this book describes an algorithm as a step-by-step procedure having threee important properties:

#### Definiteness
Every step must be precise.

Bad:
```
Do the calculation.
```

Good:
```
Add the value in R1 to the value in R2.
```

#### Effective computability
Every step must actually be something the computer can perform.

for example:
```
understand what the user wants.
```

is not directly a machine operation.

But:
```
Read a charecter.
Compare it with 'A'.
```

can be implemented.

#### Finiteness
the algorithm must actually terminate

For example:
```
1. Read number.
2. Add 1.
3. Repeat forever.
```

is a procedure, but it doesn't terminate.