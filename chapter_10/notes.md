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