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