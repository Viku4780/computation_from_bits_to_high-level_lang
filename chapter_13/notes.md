## What is a "control structure"?
let's start with problem.

Suppose i write:
```
int x = 10;

printf("A\n");
printf("B\n");
printf("C\n");
```

The computer basically does:
```
A
 ↓
B
 ↓
C
```

This is sequential execution.

The CPU executes one instructions, then another, then another.

But real programs need much more than that.

We need to say things like:

    If the user entered the correct password, allow access.

or:

    Keep asking until the user enters a valid number.

or:

    Do this 100 times.

or:

    if the operation is +, add. if it is -, subtract.

This means the program cannot simply move straight downward.

it needs to change the path of execution.

That is the fundamental idea behind control structure.


### The core abstraction

#### 1. Selection
Choose which path to execute.

#### 2. Iteration
Choosee whether to execute a path again.


### Control structures are really branches
When you write:
```
if (x < 10){
    printf("small");
}
```

The CPU doesn't have an instruction called:
```
IF
```

at the fundamental machine level.

Instead, the compiler turns your high-level idea into something resembling:
```
calculate x < 10
       |
       v
   condition?
    /     \
 false    true
  |         |
  |         v
  |       execute body
  |         |
  └─────────┘
```

At the machine level, this becomes comparisons + conditional branches/jumps.

This is the bridge:
```
C
│
├── if
├── while
├── for
├── do-while
├── switch
├── break
└── continue
       ↓
control flow
       ↓
conditional/unconditional branches
       ↓
machine instructions
       ↓
CPU changes PC
       ↓
different instruction executes
```

Remember your earlier understanding of the program counter (PC).

The PC normally says:

    Which instruction should execute next?

A control structure essentially changes the answer to:

    Which instruction should execute next?

That is the essence

### Before learning if: C's idea of true and false

In C:
```
0         -> false
non-zero  -> true
```

    C doesn't require a special Boolean value for every condition. Zero means false; anything non-zero means true.


### Where do Boolean-looking conditions come from?
Usually from relational operators.
```
<
>
<=
>=
==
!=
```

Conceptually:
```
5 < 10
 ↓
true
 ↓
non-zero value
```

if:
```
x = 15;
```

then:
```
15 < 10
 ↓
false
0
```

so:
```
if (x < 10)
```
is essentially asking:

    Does this expression produce a non-zero value?

### if - the first decision strucutre
The basic syntax:
```
if (condition)
    statement;
```

Example:
```
int age = 20;

if (age >= 18)
    printf("Adult\n");
```

The CPU conceptual behavior is:
```
evaluate age >= 18
        |
        v
      true?
     /     \
   no       yes
   |         |
   |         v
   |      printf
   |         |
   └─────────┘
```

if false, it simply skips the statement.

### Why braces {} matter
You can write:
```
if (x > 10)
    printf("A\n");
```

But if you need multiple statements:
```
if (x > 10){
    printf("A\n");
    printf("B\n");
    x = 0;
}
```

The braces make these three statements one compound statement, or block.

Think:
```
if condition
    execute ONE statement
```

A block lets that "one statement" become:
```
{
    statement 1
    statement 2
    statement 3
}
```

so:
```
if (x > 10) {
    A;
    B;
    C;
}
```

means:

    If true, execute the entire block.

### The dangerous mistake
Look carefully:
```
if (x > 10)
    printf("A\n");
    printf("B\n");
```

A beginner might visually think:
```
if true:
    A
    B
```

But C sees:
```
if (x > 10)
    printf("A\n");

printf("B\n");
```

Only the next statement belongs to the if.

Therefore:
```
if true:
    A
always:
    B
```

This is why braces are strongly recommended when there is any possibility of confusion.

### if-else
Now suppose we don't merely want:
```
"Do something if true."
```

We want:
```
"Do A if true, otherwise do B."
```

That's if-else.
```
if (condition) {
    A;
}
else {
    B;
}
```

Example:
```
int age = 15;

if (age >= 18) {
    printf("Adult\n");
}
else {
    printf("Minor\n");
}
```

Conceptually:

             age >= 18?
             /        \
           yes         no
            |           |
            v           v
         Adult        Minor
            \           /
             \         /
              continue

Exactly one branch executes.

### Think of if-else as mutually exclusive paths
Given:
```
if (condition) {
    A;
}
else {
    B;
}
```

you should mentally understand:
```
condition = true
    → A

condition = false
    → B
```

Never:
```
A and B
```
Only one branch is selected.

## else if
Suppose we have three possibilities:
```
score >= 90 → A
score >= 80 → B
score >= 70 → C
otherwise   → F
```

We can write:
```
if (score >= 90) {
    printf("A");
}
else if (score >= 80) {
    printf("B");
}
else if (score >= 70) {
    printf("C");
}
else {
    printf("F");
}
```

Conceptually this is:

             score >= 90?
              /       \
            yes        no
             |          |
             A       score >= 80?
                        /      \
                      yes       no
                       |         |
                       B      score >= 70?
                                  /    \
                                yes     no
                                 |       |
                                 C       F

But here's something important:

    else if isn't a fundamentally new machine capability.

It is essentially a convenient way of writing nested decisions.

For example:
```
if (a) {
    A;
}
else if (b) {
    B;
}
else {
    C;
}
```

is conceptually equivalent to:
```
if (a) {
    A;
}
else {
    if (b) {
        B;
    }
    else {
        C;
    }
}
```
That's the kind of abstraction you should understand.

### Nested if
You can put an if inside another if.

Example:
```
if (age >= 18) {
    if (has_license) {
        printf("Can drive");
    }
}
```

Mental model:
```
age >= 18?
   |
   yes
   |
has_license?
   |
   yes
   |
Can drive
```

The inner decision is reached only if the outer decision allows execution to reach it.

This becomes useful when conditions depend on previous decisions.

## Iteration
Imagine:
```
printf("Hello\n");
printf("Hello\n");
printf("Hello\n");
printf("Hello\n");
printf("Hello\n");
```

That's repetitive.

What if we need 1,000 repetitions?

We need a loop.

the book emphasizes that iteration is fundamental to useful computation and introduces three C iteration contructs:
```
while
for 
do-while
```

```
        ┌───────────────┐
        │     TEST      │
        └───────┬───────┘
                |
          true? | false
           /    |    \
          /     |     \
         v      |      v
      BODY      |     EXIT
         |
         |
         └───────────> TEST
```

A loop always has:

1. A condition
2. Some work
3. A way to eventually make the condition false

if the third part is missing, you may create an infinite loop.

### While
Syntax:
```
while (condition) {
    body;
}
```

Example:
```
int x = 0;

while (x < 10 ) {
    printf("%d\n", x);
    x++;
}
```

Execution:
```
x = 0

x < 10?
yes
 ↓
print 0
 ↓
x = 1
 ↓
x < 10?
yes
 ↓
print 1
 ↓
x = 2
...
```

Eventually:
```
x = 10

10 < 10?
   ↓
 false
   ↓
 exit loop
```

The book explicitly describes while as testing its condition before each execution of the loop body.

### The most important property of while
A while loop can execute:
```
0 times
```
because the condition is checked first.

example:
```
int x = 20;

while (x < 10){
    printf("Hello");
}
```
The body never executes.

Why?
```
20 < 10
 ↓
false
 ↓
exit immediately
```
This is called a pre-test loop.

### Infinite loop
Look at this:
```
int x = 0;

while(x < 10){
    printf("%d\n", x);
}
```

What's wrong?

x never changes.

So:
```
x = 0
x < 10 → true
print
x = 0
x < 10 → true
print
x = 0
...
```

forever.

    Every loop needs a believable path toward termination, unless you intentionally want an infinite loop.


### Sentinel-controlled loops
Sometimes you don't know beforehand how many times the loop should execute.

For example:

    Keep reading input until the user enters -1.

Then:
```
int x;

scanf("%d", &x);

while(x != -1){
    printf("You entered %d\n", x);
    scanf("%d", &x);
}
```

Here -1 is sentinel.

it means:

    Stop.

So:
```
read input
   ↓
is it -1?
 /     \
yes     no
 |       |
stop    process
          |
          └── read again
```

The book specifically associates while with sentinel-controlled iteration, where the number of iterations isn't known beforehand.


### for
Now suppose we know how many times we want to repeat something.

Example:
```
Print numbers 0 through 9.
```

You could write:
```
int x = 0;

while (x < 10) {
    printf("%d\n", x);
    x++;
}
```

But C gives us a compact form:
```
for (int x = 0; x < 10; x++) {
    printf("%d\n", x);
}
```
The book describes for as particularly suitable for counter-controlled loops.

### Understand the three parts of for
This:
```
for (initialization; condition; update)
```

contains three pieces.

For example:
```
for (int x = 0; x < 10; x++)
```

means:

Initialization
```
int x = 0
```

Do this once.

Condition
```
x < 10
```

Check before every iteration.

Update
```
x++
```

Do this after each iteration.

So:
```
initialization
      |
      v
   condition
    /     \
 false     true
  |          |
 exit       body
             |
             v
           update
             |
             └──────> condition
```
This diagram is worth remembering.

### for is not magic
This is one of the biggest abstractions to remove.

This:
```
for (int x = 0; x < 10; x++) {
    printf("%d\n", x);
}
```
is conceptually equivalent to:
```
int x = 0;

while (x < 10) {
    printf("%d\n", x);
    x++;
}
```
The for syntax is mainly a convenient way to express this common pattern.

The book explicitly points out that a for loop can be constructed using a while loop and vice versa.

So don't think:

    "for is a completely different kind of machine operation."

Think:

    "for is another way of expressing iteration."

### A subtle for trap: the semicolon
Look at:
```
for (x = 0; x < 10; x++);
```

There is a semicolon immediately after the for.

That means the loop body is an empty statement.

So:
```
for (x = 0; x < 10; x++);
sum = sum + x;
```

does not mean:
```
for (...) {
    sum = sum + x;
}
```
Instead:
```
loop does nothing
x becomes 10
then:
sum = sum + x
```
The book uses this exact type of mistake to demonstrate how easy it is to accidentally create an empty loop body.

### Scope of a for variable
You can write:
```
for (int i = 0; i < 10; i++) {
    printf("%d\n", i);
}
```
Here i is declared inside the for.

Its scope is essentially the for statement and its body.

So after the loop:
```
printf("%d", i);
```
is not valid because i no longer exists in that scope.