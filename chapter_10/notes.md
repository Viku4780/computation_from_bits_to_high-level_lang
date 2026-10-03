## Why can't we just add the keyboard values?
This is the problem the chapter deliberately demonstrates first.

Suppose you type:
```
2
3
```

The keyboard gives:
```
'2' = x32
'3' = x33
```

if the program simply does:
```
ADD R0,R1,R0
```

Then it calculates:
```
x32 + x33
```

which is:
```
x65
```

and x65 is the ASCII code for:
```
e
```

so you get:
```
2 + 3 = e
```

### Why did 2 + 3 = e happen?
Because there are three different concepts being confused.

You typed:
```
2
```

The keyboard gives:
```
ASCII '2'
=
x32
```

But arithmetic needs:
```
integer 2
=
x0002
```

Those are different bit patterns.

Likewise:
```
ASCII '3'
=
x0033
```

while:
```
integer 3
=
x0003
```

So:
```
x0032 + x0033
=
x0065
```

and the monitor interprets:
```
x0065
```

as an ASCII character:
```
'e'
```

The actual problem is therefore:
```
ASCII
   ≠
integer representation
```

## the two data types used by the calculator
the calculator mainly works with two representations.

### representation 1 - ASCII strings
used for:
```
keyboard input 
monitor output
```

examples:
```
"25"
"793"
"-42"
```

At the LC-3 level, the examples store each ASCII character in its own 16-bit word to simplify the algorithms.


### Representation 2 - 2's-complement integers
used for:
```
arithmetic
```

examples:
```
25
-42
793
```

represented in binary using two's complement.

so the calculator is constantly performing:
```
ASCII -> binary
```

and:
```
binary -> ASCII
```

### Why data type conversion matters
Imagine the expression:
```
A + B
```

where:
```
A = ASCII string
B = binary integer
```

The ALU cannot simply receive:
```
"25"
```

and know that it means the integer 25.

The data has to be in the representation expected by the hardware doing the operation.

The textbook makes the general point:

    An operation must receive operands in the representation appropriate for that operation.

This is the machine-level foundation of many conversions that a compiler normally hides from you.

### ASCII-to-binary conversion
Now suppose the user types:
```
295
```

The keyboard provides:
```
'2' → x32
'9' → x39
'5' → x35
```

The program stores these characters in:
```
ASCIIBUFF
```

Conceptually:
```
ASCIIBUFF:

x32
x39
x35
```

and:
```
R1 = 3
```

meaning:
```
there are 3 digits
```

The chapter restricts this particular conversion routine to:
```
0 through 999
```


### First trick: remove the ASCII part
ASCII digits have a useful property.

The ASCII codes are:
```
'0' = x30
'1' = x31
'2' = x32
'3' = x33
...
'9' = x39
```

Notice the low four bits:
```
x30 → 0000
x31 → 0001
x32 → 0010
x33 → 0011
...
x39 → 1001
```

So for a valid decimal digit:
```
ASCII digit & x000F
```

gives its numerical digit value.

For example:
```
x35
AND x000F
───────
x0005
```

Thus:
```
ASCII '5'
    ↓
x35
    ↓
AND x000F
    ↓
5
```

The chapter uses exactly this technique.


### But getting individual digits isn't enough
Suppose we have:
```
2
9
5
```

After removing the ASCII template, we get:
```
2
9
5
```

But if we simply add:
```
2 + 9 + 5
```

we get:
```
16
```

not:
```
295
```

Why?

Because position matters.

The digits represent:
```
2 hundreds
9 tens
5 ones
```

Therefore:
```
295
=
2 × 100
+ 9 × 10
+ 5 × 1
```

So the conversion algorithm has to account for place value.


### Lookup tables
The textbook's routine uses lookup tables.

One table is:
```
LookUp10

0
10
20
30
40
50
60
70
80
90
```

Another:

LookUp100
```
0
100
200
300
400
500
600
700
800
900
```

So suppose the tens digit is:
```
7
```

The program uses:
```
7
```

as an index into:
```
LookUp10
```

and obtains:
```
70
```

Likewise:
```
2
```

in the hundreds table gives:
```
200
```

The chapter uses these tables because the calculator only accepts up to three digits.

### Walk through 295
Let's do the whole thing.

input:
```
"295"
```

ASCII:
```
x32 x39 x35
```

#### Ones
take:
```
x35
```

strip ASCII:
```
5
```

result accumulator:
```
R0 = 5
```

#### tens
take:
```
x39
```

strip ASCII:
```
9
```

use tens table:
```
LookUp10[9]
=
90
```

Add:
```
R0 = 5 + 90
```

So:
```
R0 = 95
```

#### hundreds
take:
```
x32
```

strip ASCII:
```
2
```

use hundreds table:
```
LookUp100[2]
=
200
```

then:
```
R0 = 95 + 200
```

so:
```
R0 = 295
```

Done

### Why start from the rightmost digit?
The routine begins with the ones digit.

This makes the indexing and accumulation straightforward.

For:
```
295
```

we can think:
```
5 → 5
9 → 90
2 → 200
```

then:
```
5 + 90 + 200
=
295
```

The chapter's flowchart and code implement exactly this staged contribution method.


### Why did the textbook allocate four words?
Initially you might think:
```
295
```

requires only:
```
3 words
```

And for input, that's true.

But later the calculator has to display results such as:
```
-295
```

So it needs:
```
sign
hundreds
tens
ones
```

Therefore:
```
4 words
```

are allocated for ASCIIBUFF.

This is a good example of designing a data structure based on all future uses, not just the first use.


### The chapter raises an important algorithm question
The lookup-table method works for:
```
000 → 999
```

But imagine:
```
123456789
```

Would we need:
```
thousands table
ten-thousands table
hundred-thousands table
...
```

A better general algorithm exists.

The book deliberately points you toward that question as an exercise.

And the general mathematical idea is:
```
value = value × 10 + next_digit
```

For example:
```
12
```

then add 3:
```
12 × 10 + 3
=
123
```

This eliminates the need for separate place-value lookup tables.

That particular generalization is posed by the textbook as a challenge rather than used in its three-digit implementation.


### Binary-to-ASCII conversion
Now let's go the other direction.

Suppose arithmetic produces:
```
295
```

The monitor doesn't want:
```
binary 295
```

It wants:
```
'2'
'9'
'5'
```

So we need:
```
binary integer
      ↓
decimal digits
      ↓
ASCII characters
```

The textbook's routine supports values:
```
-999 through +999
```


### First determine the sign
Suppose:
```
R0 = -295
```

The routine tests the sign using the condition codes.

If negative:
```
store '-'
```

If nonnegative:
```
store '+'
```

The ASCII codes used are:
```
'+' = x2B
'-' = x2D
```

Then for a negative number it converts the value to its absolute magnitude using two's-complement negation:
```
NOT R0,R0
ADD R0,R0,#1
```

So:
```
-295
```

becomes:
```
295
```

internally.


### Why is absolute value convenient?
Because extracting decimal digits is easier when we don't have to worry about the sign during every step.

So the routine effectively does:
```
-295
 ↓
sign = '-'
 ↓
magnitude = 295
 ↓
extract digits
```

rather than trying to extract digits from the negative representation directly.


### Finding the hundreds digit
Suppose:
```
R0 = 295
```

The algorithm repeatedly subtracts:
```
100
```

until the result would become negative.

Let's trace it:
```
295 - 100 = 195
195 - 100 = 95
95 - 100 = -5
```

We successfully subtracted 100:
```
2 times
```

so the hundreds digit is:
```
2
```

But we've gone one subtraction too far.

The current value is:
```
-5
```

So the algorithm corrects by adding 100 back:
```
-5 + 100 = 95
```

Now:
```
hundreds = 2
remainder = 95
```

The textbook explicitly describes this repeated-subtraction-and-correction strategy.

### Finding the tens digit
Now we have:
```
95
```

Again subtract 10:
```
95 - 10 = 85
85 - 10 = 75
...
15 - 10 = 5
5 - 10 = -5
```

We successfully subtracted:
```
9 times
```

so:
```
tens = 9
```

Then correct:
```
-5 + 10 = 5
```

Remaining:
```
5
```

Therefore:
```
ones = 5
```

Result:
```
295
```

### Convert each digit to ASCII
Now we have numerical digits:
```
2
9
5
```

To convert a decimal digit to ASCII:
```
ASCII = digit + x30
```

So:
```
2 + x30 = x32
9 + x30 = x39
5 + x30 = x35
```

Therefore:
```
295
```

becomes:
```
x32 x39 x35
```

which is:
```
"295"
```

### The complete conversion pipeline
Now you should see:
```
Keyboard
   ↓
ASCII characters

'2' '9' '5'
   ↓
ASCII → binary
   ↓
295
   ↓
arithmetic
   ↓
some result
   ↓
binary → ASCII
   ↓
'2' '9' '5'
   ↓
Monitor
```

This is the fundamental data flow of the calculator.


## Normal LC-3 arithmetic
The LC-3 is a three-address machine for the purpose being discussed.

For example:
```
ADD R0,R1,R2
```

means:
```
R0 = R1 + R2
```

There are explicitly:
```
source 1
source 2
destination
```

So:
```
ADD R0,R1,R2
```

contains three operand locations.

### Two-address machines
Other processors use something like:
```
ADD EAX,EBX
```

The result overwrites one of the operands.

Conceptually:
```
EAX = EAX + EBX
```

So only two explicit locations are specified.

The chapter contrasts this with the LC-3


### Stack machines
Now imagine we don't specify operands at all.

Instead:
```
stack:
42
17
```

and the instruction simply says:
```
ADD
```

The machine knows:

    "Take the top two values."

So:
```
POP → 17
POP → 42
42 + 17 = 59
PUSH 59
```

The instruction didn't specify:
```
R0
R1
R2
```

The stack determines the operands.

Such an architecture is commonly called a:
### Zero-address machine

because an operation like:
```
ADD
```

doesn't explicitly specify operand addresses.

### Why is this convenient for a calculator?
Because a person can press:
```
+
```

and conceptually that means:
```
take the last two values
add them
store result
```

The user doesn't need to tell the machine:
```
"Use R1 and R4 and put result in R2."
```

The stack already determines where the operands are.

### Example: (25 + 17) × (3 + 2)
The chapter uses:
```
(A + B) × (C + D)
```

with:
```
A = 25
B = 17
C = 3
D = 2
```

Let's see the stack.

#### Step 1
```
PUSH 25
```

Stack:
```
25
```

#### Step 2
```
PUSH 17
```

Stack:
```
17 ← top
25
```

#### Step 3
```
ADD
```

Pop:
```
17
25
```

Add:
```
25 + 17 = 42
```

Push:
```
42
```

Stack:
```
42
```

#### Step 4
```
PUSH 3
```

Stack:
```
3
42
```

#### Step 5
```
PUSH 2
```

Stack:
```
2
3
42
```

#### Step 6
```
ADD
```

Pop:
```
2
3
```

Calculate:
```
3 + 2 = 5
```

Push:
```
5
42
```

#### Step 7
```
MULTIPLY
```

Pop:
```
5
42
```

Calculate:
```
42 × 5 = 210
```

Push:
```
210
```

#### Step 8
```
POP
```

Result:
```
210
```

That's exactly the eight-step calculator-style computation described by the chapter.


## The three arithmetic operations
The calculator needs:
```
OpAdd
OpMult
OpNeg
```

They correspond to:
```
+
*
-
```

But there is a subtle point.

The calculator's - operation is unary negation.

It doesn't directly mean:
```
A - B
```

Instead:
```
A - B
```

is performed as:
```
A + (-B)
```

because:
```
A - B = A + (-B)
```

### OpNeg
Suppose the stack is:
```
7
```

OpNeg does:
```
POP
```

giving:
```
7
```

Then two's-complement negation:
```
NOT R0,R0
ADD R0,R0,#1
```

produces:
```
-7
```

Then:
```
PUSH -7
```

So:
```
7
 ↓
OpNeg
 ↓
-7
```

The textbook's OpNeg routine follows exactly this pattern.


### OpAdd
Now let's understand OpAdd.

Its job is:
```
POP first operand
POP second operand
ADD
RangeCheck
PUSH result
```

But there is an important concern:

    What happens when something goes wrong?

There are two major failures.

### OpAdd failure #1 — not enough operands
Suppose stack contains only:
```
10
```

and the user presses:
```
+
```

The routine performs its first POP:
```
10
```

Success.

Then it performs its second POP.

But the stack is empty.

So the second POP fails.

Now the routine must return the stack to its original state.

Remember:

Before OpAdd:
```
10
```

After first POP:
```
empty
```

After second POP fails:
```
empty
```

That would be wrong.

So OpAdd adjusts the stack pointer to make the first operand available again.

The chapter explicitly emphasizes this rollback behavior.

### OpAdd failure #2 — result too large
Suppose:
```
700
500
```

are on the stack.

Add:
```
700 + 500 = 1200
```

But the calculator only accepts:
```
-999 through +999
```

So the result cannot be accepted.

What should happen?

The calculator should not simply lose both operands.

It restores them to the stack.

Original:
```
500
700
```

After the failed operation:
```
500
700
```

again.

That's good error-handling design.

### This is a very important programming lesson
The routine isn't merely:
```
do the operation
```

It is:
```
attempt operation
     ↓
check whether input is valid
     ↓
perform operation
     ↓
check whether result is valid
     ↓
commit result
```

or:
```
failure
 ↓
rollback
```

That is a powerful general programming pattern.

### RangeCheck
The calculator wants all values to stay between:
```
-999
```

and:
```
+999
```

So there is a helper subroutine:
```
RangeCheck
```

Suppose:
```
R0 = 500
```

Then:
```
500 <= 999
500 >= -999
```

so:
```
R5 = 0
```

meaning:
```
success
```

If:
```
R0 = 1200
```

then:
```
1200 > 999
```

so:
```
R5 = 1
```

meaning:
```
failure
```

The routine also prints an error message when out of range.

### Why use negative constants for comparison?
Remember the LC-3 doesn't have:
```
CMP R0,#999
```

So the program uses subtraction through addition.

To check:
```
R0 > 999
```

it can form:
```
R0 + (-999)
```

If the result is positive:
```
R0 > 999
```

Similarly, to check the lower limit:
```
R0 + 999
```

If that becomes negative:
```
R0 < -999
```

This is another example of using the primitive LC-3 operations creatively.

### The LC-3 has no MUL instruction
This becomes important for OpMult.

We cannot simply write:
```
MUL R0,R1,R2
```

because the LC-3 ISA being used doesn't provide a multiply instruction.

So how do we multiply?

We already learned the basic algorithm:
```
A × B
```

can be computed as:
```
A + A + A + ... + A
```

B times.

### But negative numbers create a problem
Suppose:
```
6 × -4
```

We can't simply repeat the addition -4 times.

So the algorithm separates:
```
magnitude
```

from:
```
sign
```

### The sign flag
The OpMult routine uses a register as a sign flag.

Conceptually:
```
flag = 0
```

means:
```
multiplier positive
```

and:
```
flag = 1
```

means:
```
multiplier negative
```

If the multiplier is negative:
```
remember negative sign
negate multiplier
```

Now we can perform repeated addition using the positive magnitude.

### Example: 6 × -4
Initially:
```
multiplicand = 6
multiplier = -4
```

Detect:
```
multiplier negative
```

Set flag:
```
negative
```

Then two's-complement negate:
```
-4 → 4
```

Now calculate:
```
6 + 6 + 6 + 6
=
24
```

Finally, because the flag says the final result should be negative:
```
24 → -24
```

Then push:
```
-24
```

The textbook's OpMult routine follows this logic.

### Why only the multiplier's sign matters?
Because:
```
(+A) × (+B) = positive
(+A) × (-B) = negative
(-A) × (+B) = negative
(-A) × (-B) = positive
```

The sign can be handled separately from the magnitude.

The routine chooses one operand as the multiplier, tracks its sign, converts it to a positive count, and applies the corresponding final sign.

### What if the multiplier is zero?
Suppose:
```
A × 0
```

The answer is immediately:
```
0
```

The routine detects this and avoids unnecessary repeated addition.

This is a nice example of handling a special case efficiently.

### What happens if multiplication overflows?
Suppose:
```
500 × 3
=
1500
```

But valid range is:
```
-999 ... +999
```

So:
```
1500
```

is invalid.

The result is rejected and the original operands are restored to the stack.

Again:
```
attempt
 ↓
validate
 ↓
commit only if valid
```

The OpMult routine is designed around this same rollback idea as OpAdd.

### Now build the calculator itself
We have all the pieces:
```
keyboard input
      ↓
GETC
      ↓
ASCII characters
      ↓
PushValue
      ↓
ASCIItoBinary
      ↓
PUSH
      ↓
STACK
      ↓
OpAdd / OpMult / OpNeg
      ↓
STACK
      ↓
OpDisplay
      ↓
POP
      ↓
BinarytoASCII
      ↓
PUTS
      ↓
monitor
```

The calculator is essentially a manager coordinating these pieces.

## An important subtlety: this is basically stack notation
The calculator doesn't behave like a normal mathematical calculator where you necessarily type:
```
25 + 17
```

and press equals.

Instead, you put values onto the stack and operations consume those values.

For example:
```
25
17
+
```

means:
```
PUSH 25
PUSH 17
ADD
```

Result:
```
42
```
This is essentially postfix / reverse-polish style evaluation.

The textbook's larger example uses this stack-oriented command sequence.

### Evaluate a more complex expression
The textbook demonstrates:
```
(51 - 49) * (172 + 205) - (17 * 2)
```

which gives:
```
720
```

The user enters:
```
51 LF
49 LF
-
172 LF
205 LF
+
*
17 LF
2 LF
*
-
+
D
```
Let's understand the stack evolution.

### First part: 51 - 49
After:
```
51
49
```
stack:
```
49
51
```
Press:
```
-
```
That means:
```
negate 49
```
stack:
```
-49
51
```
Then the next + operation:
```
51 + (-49)
=
2
```
stack:
```
2
```
So:
```
51 - 49 = 2
```

### Second part: 172 + 205
Input:
```
172
205
```

stack:
```
205
172
2
```

Press:
```
+
```

gives:
```
377
```

stack:
```
377
2
```

### Multiply
Press:
```
*
```
Now:
```
2 × 377 = 754
```

stack:
```
754
```

### Third part: 17 * 2
Enter:
```
17
2
```

stack:
```
2
17
754
```

Press:
```
*
```

gives:
```
34
```

stack:
```
34
754
```

### Subtract 34 from 754
Press:
```
-
```

This is unary negation:
```
34 → -34
```

Stack:
```
-34
754
```

Then:
```
+
```

gives:
```
754 + (-34)
=
720
```

Stack:
```
720
```

### Display
Press:
```
D
```

The calculator:
```
POP
```

gets:
```
720
```

Then:
```
BinarytoASCII
```

produces something like:
```
"+720"
```

Then:
```
PUTS
```

prints it.

Importantly, the display routine then pushes the number back onto the stack, so displaying it does not destroy the calculator's stored value. The textbook's OpDisplay routine explicitly performs this restore-to-stack behavior after displaying the result.


### C — clear the stack
How do we clear the calculator stack?

We don't have to individually POP every element.

Remember the fundamental stack rule:
```
R6 = stack pointer
```

So the empty stack is represented by:
```
R6 = StackBase + 1
```

Therefore the clear operation simply resets:
```
R6
```
to the empty-stack position.

This is one of the nicest examples of the abstraction:

    We don't have to erase every old memory word. We only have to restore the pointer that defines the logical stack.

The textbook's OpClear routine does exactly that.

### This is exactly like Chapter 8
Remember our earlier point:
```
physical memory
        ≠
logical structure
```

Suppose:
```
memory still contains:

720
34
754
```

but:
```
R6 = empty position
```

Then logically:
```
stack = empty
```

The old memory values don't matter.

Again:
```
representation
     +
control information
     ↓
logical structure
```

### Let's examine the calculator's main algorithm
The main program begins by setting the stack pointer:
```
LEA R6,StackBase
ADD R6,R6,#1
```

So:
```
R6 = StackBase + 1
```

which means:
```
empty stack
```

Then it repeatedly:
```
prompt
get command
figure out what command it is
perform corresponding action
return to prompt
```


### Command dispatch
Suppose the user enters:
```
+
```

The main program essentially performs:

Is it X?
```
No.
```

Is it C?
```
No.
```

Is it +?
```
Yes.
```

Call OpAdd.

Then returns to:
```
NewCommand
```

This is a simple form of a command dispatcher.

Conceptually:

             input character
                    |
        +-----------+-----------+
        |           |           |
        X           C           +
        |           |           |
       halt       clear        add

This pattern appears everywhere in software:
```
command
  ↓
identify command
  ↓
dispatch handler
```

### If it isn't a recognized command
Suppose the character isn't:
```
X
C
+
*
-
D
```

Then the program assumes:
```
The user is entering a number.
```

So it calls:
```
PushValue
```

This is a nice example of the main program separating policy from work:

main:
"What kind of thing did the user enter?"

PushValue:
"Okay, I'll process the number."

### PushValue is more complicated than simply GETC
Why?

Because a number isn't necessarily one character.

Suppose:
```
295
```

The routine must:
```
read '2'
read '9'
read '5'
read Enter
```

while checking every character.

The textbook's PushValue routine therefore performs several jobs.

### Job 1 — ensure the character is a digit
Valid:
```
'0' through '9'
```

ASCII range:
```
x30 through x39
```

So for every typed character, the routine checks:
```
character >= '0'
```

and:
```
character <= '9'
```

If not:
```
NotInteger
```

### Job 2 — don't accept more than three digits
The calculator allows:
```
maximum = 3 digits
```

So it maintains a digit count / remaining capacity.

Suppose the user types:
```
123
```

fine.

But:
```
1234
```

causes:
```
Too many digits
```

The routine then consumes input until the line is finished so the calculator can return to a clean state.

### Job 3 — detect Enter / LF
The calculator considers the end of number input to be the line-feed character:
```
x0A
```

So:
```
295
Enter
```

looks conceptually like:
```
'2'
'9'
'5'
LF
```

When the routine sees:
```
x0A
```

it knows:
```
The number is complete.
```

### What if the user presses Enter without typing a number?
Then:
```
LF
```

is encountered immediately.

There are no digits.

The calculator reports:
```
No number entered
```

instead of pushing an invalid value.

Again, validation happens before changing the calculator state.

### Input errors are handled carefully
The routine has separate cases for:
```
too many digits
not an integer
no digits
```

This is useful because the program doesn't simply crash when the input doesn't match expectations.

It responds with a controlled error message.

The chapter's PushValue routine includes these error paths.

### Stack memory layout for the calculator
The main program provides:
```
StackMax
StackBase
ASCIIBUFF
```

The stack storage is allocated in the main program.

The textbook points out an important linking/assembly issue here:

    The subroutines reference these global labels, so the entire calculator is assembled as one unit in this implementation.

### Why can't each subroutine simply be assembled separately?
Consider:
```
ASCIItoBinary
```

It refers to:
```
ASCIIBUFF
```

But ASCIIBUFF is defined somewhere else.

So if you assemble only:
```
ASCIItoBinary
```
there is no address yet for:
```
ASCIIBUFF
```

Hence the assembler doesn't have enough information.

The textbook notes that .EXTERNAL could be used to enable separate assembly, but instead the chapter chooses to assemble the whole calculator as a single unit.

### Why must the labels have unique names?

Because all the routines are being assembled together.

Imagine two subroutines both contain:
```
DONE
```

Which DONE does another instruction mean?
```
Ambiguous.
```

So the routines use names like:
```
AtoB_Done
BtoA...
OpAdd_...
OpMult_...
OpNeg_...
PushValue_...
```

This is a practical lesson about symbol tables and global naming.

### This is another Chapter 7 connection

Chapter 7 taught:
```
labels
 ↓
symbol table
 ↓
addresses
 ↓
machine code
```

Chapter 10 now has multiple routines referring to shared symbols.

So the earlier lesson becomes practical:
```
main program
      +
subroutines
      +
global labels
      ↓
one executable program
```

Nothing in the computer works in isolation.

### Let's understand PUSH in the calculator
The calculator's PUSH routine is slightly different in presentation from the Chapter 8 version.

It checks:
```
Is the stack full?
```

If not:
```
R6--
memory[R6] = R0
```

If full:
```
print "Error: Stack is Full."
R5 = 1
```

Otherwise:
```
R5 = 0
```

The PUSH routine also saves and restores the register it needs internally.

### POP in the calculator
POP checks:
```
Is the stack empty?
```

If empty:
```
print "Error: Too Few Values on the Stack."
R5 = 1
```

Otherwise:
```
R0 = memory[R6]
R6++
R5 = 0
```

So every arithmetic routine gets a useful interface:
```
POP
 ↓
R0 = value
R5 = success/failure
```

### Why does every subroutine save registers?
Remember Chapter 8's caller/callee-save discussion.

For example, OpAdd saves:
```
R0
R1
R5
R7
```

before doing its own work.

Why?

Because it is a callee and does not want to unexpectedly destroy values that the caller might need.

Likewise, OpMult saves the registers it needs.

This is a real example of the save/restore convention you already learned.

### Notice the role of R7
Every time a subroutine does:
```
JSR
```

R7 receives a return address.

But many calculator routines themselves call other routines.

For example:
```
OpAdd
  ↓
POP
  ↓
RangeCheck
  ↓
PUSH
```

Therefore R7 can be overwritten.

That's why a routine such as OpAdd saves its own R7 before making calls.

This is exactly the nested-call problem from Chapter 8.


```
Calculator
    |
    +-- GETC
    |
    +-- OUT
    |
    +-- PushValue
    |      |
    |      +-- ASCIItoBinary
    |      |
    |      +-- PUSH
    |
    +-- OpAdd
    |      |
    |      +-- POP
    |      +-- RangeCheck
    |      +-- PUSH
    |
    +-- OpMult
    |      |
    |      +-- POP
    |      +-- RangeCheck
    |      +-- PUSH
    |
    +-- OpNeg
    |      |
    |      +-- POP
    |      +-- PUSH
    |
    +-- OpDisplay
           |
           +-- POP
           +-- BinarytoASCII
           +-- PUTS
           +-- restore value
```

```
                     CALCULATOR
                         |
                 +-------+-------+
                 |               |
              INPUT            COMMAND
                 |               |
               GETC          dispatcher
                 |
            ASCII string
                 |
          +------+------+
          |             |
       validate      convert
          |             |
          +------> ASCIItoBinary
                         |
                       binary
                         |
                       PUSH
                         |
                         ↓
                    +---------+
                    |  STACK  |
                    +---------+
                         |
             +-----------+-----------+
             |           |           |
           ADD         MULT        NEG
             |           |           |
             +-----------+-----------+
                         |
                       STACK
                         |
                      DISPLAY
                         |
                         ↓
                  BinarytoASCII
                         |
                       PUTS
                         |
                         ↓
                      MONITOR
```

### Why use a stack instead of registers?
Suppose you're evaluating a complicated expression.

With registers, you have to manage:
```
R0
R1
R2
R3
...
```

and carefully remember which value is in which register.

With a stack:
```
PUSH value
PUSH value
operation
```

The ordering itself carries the operand information.

So the program can treat the stack as temporary expression storage.