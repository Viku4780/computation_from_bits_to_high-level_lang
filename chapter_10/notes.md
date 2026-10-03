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