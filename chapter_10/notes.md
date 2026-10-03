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