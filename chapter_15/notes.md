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


## Why testing exists
The textbook describes testing as putting software through synthetic trials:
```
provide inputs
     ↓
observe behavior
     ↓
compare against expected behavior
     ↓
discover bugs
```
Real software may undergo huge numbers of trials before release.

So:
 
    Testing is not simply "run the program and see if it works."

it is a systematic attempt to find situation in which the program violates its requirements.


### The perfect testing problem
ideally, we'd test:

    Every possible input.

But that's usually impossible.

Suppose:
```
A = 32-bit integer
B = 32-bit integer
```

There are enormous numbers of possible combinations.

the testbook points out that exhaustive testing can become computationally impossible; even at one million tests per second, testing evey pair of 32-bit values would take roughly half million years

so we need:

    Intelligent test selection.

### Test cases
A test case is essentially:
```
input
+
expected behavior/output
```

For example, suppose:
```
int Square(int x);
```
Test case:
```
input: 5
expected: 25
```
Another:
```
input: 0
expected: 0
```
Another:
```
input: -5
expected: 25
```
Each one tests some aspect of behavior.

### Don't only test normal inputs
Suppose you have:
```
int Divide(int a, int b);
```

You might test:
```
10 / 2 = 5
```

Good.

But what about:
```
0 / 2
10 / 1
10 / 0
-10 / 2
10 / -2
INT_MAX / 1
```

This is where good testing becomes interesting.

You want inputs that expose weaknesses.

### Boundary values
One of the most useful testing ideas is:

    Test the boundaries.

Suppose the valid range is:
```
1 through 100
```

Don't only test:
```
50
```

Test:
```
0
1
2
99
100
101
```

Why?

Because bugs often occur around boundaries.

For example:
```
if (score > 50)
```
versus:
```
if (score >= 50)
```
The difference only becomes visible around:
```
50
```

### Why boundaries are powerful
Imagine:
```
if (age >= 18)
{
    printf("Adult");
}
```

Important test values:
```
17
18
19
```

Why?

Because the condition changes behavior at 18.

So:
```
boundary - 1
boundary
boundary + 1
```
is a useful testing pattern.

### Black-box testing
In black-box testing, we don't care how the program is implemented.

We care about:
```
INPUT → PROGRAM → OUTPUT
```

We treat the program as a black box:
```
                 ┌─────────────┐
input ──────────►│             │──────────► output
                 │ BLACK BOX   │
                 │             │
                 └─────────────┘
```
We don't inspect the internal code.

We simply ask:

    Given this input, does the program produce the correct output?

The textbook defines black-box testing as checking whether the program meets its input/output specifications while disregarding its internals.

### Example of black-box testing
Suppose:
```
int Square(int x);
```
You don't care how it's implemented.

Maybe:
```
return x * x;
```

Maybe:
```
return pow(x, 2);
```
Maybe something else.

You test:
```
Input       Expected
--------------------
0           0
1           1
2           4
5           25
-5          25
```
You're testing behavior.

That's black-box testing.


### White-box testing
Now turn the box transparent.

In white-box testing, we know the internal implementation and design tests that exercise different parts of it.

Imagine:
```
if (x > 10)
{
    ...
}
else
{
    ...
}
```
A white-box tester wants tests that execute:
```
true branch
```
and:
```
false branch
```
So perhaps:
```
x = 11
x = 10
```

The textbook describes white-box testing as targeting different facets of the implementation to provide assurance that many parts of the code are exercised.

```
|                | Black-box                     | White-box                             |
| -------------- | ----------------------------- | ------------------------------------- |
| Focus          | Behavior                      | Implementation                        |
| Code knowledge | Not required                  | Used                                  |
| Main question  | "Does it do the right thing?" | "Did we exercise the important code?" |
| Based on       | Specification                 | Program structure                     |

```

### Automated testing
imagine manually doing:
```
run program
enter input
read output
compare expected
repeat
```

100,000 times.

obviously ridiculous

instead:
```
test program
     ↓
generate input
     ↓
run
     ↓
check output automatically
     ↓
repeat
```

This is automated testing.

the textbook notes that larger programs commonly automate black-box testing so many more trials can be run per unit time.


### Test oracle
Suppose your program produces:
```
37
```

How do you know whether 37 is correct?

You need something that tells you the expected result.

That mechanism is often called a:

    test oracle

For example:
```
program under test
       │
       ├── input = 10
       │
       ▼
      37
       │
       ▼
 compare with expected = 42
       │
       ▼
     FAIL
```

The textbook discusses developing an independent checker program to automatically verify whether outputs meet specifications.


### Why "just run it" is weak testing
Suppose your program is:
```
int Divide(int a, int b)
{
    return a / b;
}
```

You run:
```
Divide(10, 2)
```

and get:
```
5
```

You say:

    "It works!"

No.

You have established only:

    "It works for this particular test case."

That's a very different claim.

Good testing asks:
```
What inputs could cause failure?
```

### Testing should be adversarial
A useful mindset is:

    Try to break your own program.

Don't merely ask:

    "Can I make it work?"

Ask:

    "Can I make it fail?"

For:
```
int Square(int x)
```

try:
```
0
1
-1
INT_MAX
INT_MIN
```

For an array:
```
empty
one element
first element
last element
out-of-range
```

For a parser:
```
empty input
very long input
invalid character
unexpected format
```
This mindset is extremely valuable in real software engineering.


### Example
Suppose:
```
int result = Calculate(10);
printf("%d\n", result);
```

Expected:
```
100
```

Actual:
```
90
```

Testing tells you:
```
FAIL
```

Now debugging begins.

You investigate:
```
Calculate()
```

Maybe:
```
int Calculate(int x)
{
    return x * 9;
}
```
There is your defect.

Testing found the symptom.

Debugging found the cause.


Good debugging is:
```
observe
→ form hypothesis
→ gather evidence
→ isolate cause
→ make targeted change
→ retest
```


## The debugger
A debugger allows us to observe program execution.

Instead of:
```
program
 ↓
runs extremely fast
 ↓
wrong result
```

we can pause:
```
program
 ↓
BREAK
 ↓
inspect state
 ↓
execute one step
 ↓
inspect again
```

The textbook explains that modern IDEs commonly provide source-level debuggers with operations for controlling execution and examining variables/memory.


### Breakpoints
A breakpoint tells the debugger:

    Stop execution when you reach this location.

Suppose:
```
int x = 10;
int y = 20;
int z = x + y;
printf("%d\n", z);
```

Place a breakpoint at:
```
int z = x + y;
```

Then execution proceeds:
```
x = 10
y = 20
      ↓
BREAKPOINT
      ↓
program pauses
```

Now you can inspect:
```
x
y
z
```

### Why breakpoints are powerful
Without a breakpoint:
```
program executes
  ↓
everything happens
  ↓
wrong output
```

with one:
```
program executes
      ↓
breakpoint
      ↓
FREEZE TIME
      ↓
inspect state
```

### Breakpoint at a function
you can often set a breakpoint not just at a line but at a function.

Suppose:
```
int Calculate(int x)
{
    return x * 10;
}
```
you can tell the debugger:
```
Stop whenever Calculate() begins.
```
Then:
```
main
 ↓
Calculate
 ↓
BREAK
```

Now you can inspect the function's parameters and state.

### Conditional breakpoints
Suppose:
```
for(x = 0; x < 100; x++)
{
    PerformCalculation(x);
}
```
But you suspect the problem occurs only when:
```
x == 16
```
Stopping 16 times manually woulb be annoying

A conditional breakpoint says:
```
Stop here only if x == 16.
```

So:
```
x = 0 -> continue
x = 1 -> continue
...
x = 15 -> continue
x = 16 -> BREAK
```

### Watchpoints
A watchpoint is different.

Instead of saying:
```
Stop at this line.
```

you can say:
```
Stop when this variable/state changes in a specified way.
```

For example:
```
LastItem == 4
```

The debugger watches the variable.

It can stop wherever the execution causes:
```
LastItem
```

to become:
```
4
```

The textbook distinguishes watchpoints from breakpoints: a breakpoint is associated with a location, whereas a watchpoint can trigger whenever a specified condition on state becomes true.


### Single-stepping
Once the debugger pauses, you can execute the program one statement at a time.

Suppose:
```
int x = 10;
int y = 20;
int z = x + y;
printf("%d\n", z);
```

Single-step:
```
Step 1:
x = 10

Step 2:
y = 20

Step 3:
z = 30

Step 4:
printf(...)
```
Instead of watching the program fly by, you're walking through it.

The textbook calls this single-stepping and emphasizes its usefulness for isolating a bug and verifying control flow.


### Why single-stepping is so powerful
Suppose you expect:
```
x:
10 → 20 → 30 → 40
```

but observe:
```
x:
10 → 20 → 999
```

Now you've narrowed down the problem.

You can ask:

    What statement executed immediately before x became 999?

### Inspecting variables
A debugger allows you to examine:
```
x
y
result
counter
array[i]
```
at a particular point in execution.

Suppose:
```
result = total / count;
```

You reach a breakpoint.

Debugger shows:
```
total = 100
count = 0
```
Now you immediately see a serious problem.

The source line itself might look reasonable.

The runtime state reveals the problem.

### Runtime state is evidence
Your source code represents:

    what you think the program should do.

Runtime state shows:

    what the program is actually doing.

Debugging compares the two.
```
EXPECTED STATE
      │
      │ compare
      ▼
ACTUAL STATE
      │
      ▼
difference
      │
      ▼
investigate
```

### The debugging loop
A disciplined debugging process looks like:
```
              ┌───────────────┐
              │ Observe bug   │
              └───────┬───────┘
                      ↓
              ┌───────────────┐
              │ Reproduce it  │
              └───────┬───────┘
                      ↓
              ┌───────────────┐
              │ Form a theory │
              └───────┬───────┘
                      ↓
              ┌───────────────┐
              │ Gather evidence│
              └───────┬───────┘
                      ↓
              ┌───────────────┐
              │ Find root cause│
              └───────┬───────┘
                      ↓
              ┌───────────────┐
              │ Fix it        │
              └───────┬───────┘
                      ↓
              ┌───────────────┐
              │ Test again    │
              └───────┬───────┘
                      │
                      └──────► if still wrong,
                               repeat
```

### Reproduce th bug
First:

    Can you make the bug happen reliably?

Suppose someone says:

    "Sometimes the program crashes."

That's not enough.

You want to determine:
```
Which input?
Which environment?
Which sequence of actions?
Which state?
```

For example:
```
Input:
age = 18
```

Steps:
```
1. create account
2. enter profile
3. save
4. logout
5. login
```

Result:
```
crash
```
Now you have a reproducible test case.

### Why reproducibility matters
If you cannot reproduce a bug:
```
change code
run
maybe works
```

you don't know whether you fixed anything.

But if:
```
same input
same steps
same failure
```

then you can test:
```
before fix → fails
after fix  → succeeds
```
That's much stronger evidence.

### Isolate the bug
Suppose the entire program has:
```
100 functions
```
and the output is wrong.

Don't inspect all 100 equally.

Narrow it down.

Maybe:
```
main
 ↓
ReadInput()
 ↓
Calculate()
 ↓
FormatOutput()
```

You discover:
```
ReadInput() correct
Calculate() wrong
FormatOutput() correct
```
Now the search space is dramatically smaller.

This is called isolation/localization of the defect.

### Binary-search-style debugging
Suppose you have:
```
1000 lines
```
and don't know where the problem is.

You can conceptually divide the program:
```
first half
second half
```
Check an intermediate result.

If the intermediate result is already wrong:
```
bug is before here
```
Otherwise:
```
bug is after here
```
Repeat.

This is essentially a binary-search mindset applied to debugging.


### Example debugging session
Suppose:
```
int Calculate(int a, int b)
{
    int x;
    int y;
    int result;

    x = a * 2;
    y = b * 2;
    result = x + y;

    return result;
}
```

Expected:

Calculate(10, 20) = 60

Actual:

50

Set breakpoint:

int Calculate(int a, int b)

Debugger:

a = 10
b = 20

Single-step:

x = 20

Correct.

Next:

y = 40

Correct.

Next:

result = 60

Correct.

Return:

60

But caller sees:

50

Now you know:

The bug probably isn't inside this calculation.

You move to the caller.

This is evidence-driven debugging.

### Debugging is not just staring at code
A common beginner mistake is:
```
read code
read code
read code
"Why doesn't this work?"
```

Better:
```
run
 ↓
observe
 ↓
inspect state
 ↓
form hypothesis
 ↓
test hypothesis
```
Programming is experimental.


## Defensive programming
The idea is:

    Don't merely react to bugs. Write programs that detect or prevent problems early.

For example:
```
if (count = 0)
{
    printf("Cannot calculate average.\n");
}
else
{
    average = total / count;
}
```

Instead of blindly:
```
average = total / count;
```
defensive programming anticipates invalid or unexpected situations.

### Preconditions
A precondition is something that must be true before an operation is valid.

For:
```
average = total / count;
```

a precondition is:
```
count != 0
```

Conceptually:
```
Before:
count != 0
    ↓
safe to divide
```

If:
```
count == 0
```
the operation isn't valid.

### Postconditions
A postcondition describes what should be true after an operation completes.

Suppose:
```
int Square(int x)
{
    return x * x;
}
```

A postcondition is:
```
result == x * x
```

You can think:
```
input satisfies preconditions
       ↓
     function
       ↓
output satisfies postconditions
```
This way of thinking makes functions much easier to reason about.


### Assertions
C provides:
```
#include <assert.h>
```

and:
```
assert(condition);
```

For example:
```
assert(count != 0);
average = total / count;
```

If the condition is false, the program can stop and report the failed assertion.

The conceptual idea is:
```
I believe this condition must be true here.
```

If it isn't:
```
STOP
Something is fundamentally wrong.
```

Assertions are especially useful for catching violations of assumptions during development.


### Why assertions help debugging
Suppose you have:
```
result = array[index];
```

You assume:
```
0 <= index < size
```

Instead of letting the program eventually crash somewhere mysterious, you can establish the assumption explicitly during development:
```
assert(index >= 0);
assert(index < size);
```

Now the failure occurs closer to the source of the problem.

This is a powerful debugging principle:

    Fail close to the cause rather than far away from it.

### The difference between prevention and debugging
Think:
```
Defensive programming
       ↓
prevent/detect bad state early
```

while:
```
Debugging
       ↓
investigate a failure that occurred
```
Good engineering uses both.