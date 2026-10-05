The central idea of this chapter:

    C lets you describe what you want the computer to do at a much higher level, while the compiler ultimately translates that description into machine instructions.


## Why do we even need C?
Imagine writing this simple idea:

    Ask the user for a number and count down to zero.

In C:
```
scanf("%d", &startPoint);

for(counter = startPoint; counter >= 0; counter--)
    printf("%d\n", counter);
```

In assembly, you have to worry about much more:

- Which register contains the number?
- Where is the counter stored?
- How do i load it?
- How do i compare it to zero?
- How do i branch?
- How do i decrement it?
- How do i print it?
- How do i call the appropriate system routine?
- which registers must i preserve?

C hides much of that mechanical work.

That is the primary purpose of a high-level language:

    Increase programmer productivity.

The book explains that high-level languages make programming easier by giving programmers human-friendly abstractions over the bits, memory, and machine operations that the hardware actually works with.


### What does "high-level" actually mean?
it does not mean:

    The computer understands human language

it means:

    The language is farther away from the physical details of the hardware

Compare:

#### Machine language
```
0001001010000011
```
you have to think in bits.

#### Assembly
```
ADD R1,R2,R3
```

Much easier

you now have:
- meaningful operations names
- register names
- labels
- symbolic addresses

#### C
```
total = price + tax;
```

Now the programming language lets you think in terms of values and operations, rather than registers and invidual instructions.

So there is a ladder:
```
Machine code
    ↓
Assembly
    ↓
C
    ↓
More abstract languages
```

the higher you go, the less you manually manage the underlying hardware.

But there is an important truth:

    The CPU has not become more intelligent. You have simply moved some of the work into the translation software.