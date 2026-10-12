## Start with the central question: how does a program communicate?
imagine this C program:
```
include <stdio.h>

int main(void)
{
    int age;

    printf("Enter your age: ");
    scanf("%d", &age);

    printf("Your age is %d\n", age);

    return 0;
}
```

You type 20, press Enter, and see the result. But let's look at what the program actually needs to do.

![alt text](image.png)

the crucial idea is that your program does not magically receive an integer just because you typed digits.

you type characters. The input mechanism makes those characters available. scanf() interprets them according to the format string, converts them to a value, and stores that value in memory.

Similarly, printf() does not simply display a variable's memory. it interprets a format string, converts values into a textual representation, and sends that text to an output stream.

This is the  foundation of I/O in C.


### The C standard library: Where I/O functions come from
Before we examine the four main functions, connect this chapter to what you learned about compilation and linking.

Consider:
```
#include <stdio.h>

int main(void)
{
    putchar('A');
    return 0;
}
```

you wrote putchar('A'), but you did not write the implementation of putchar() yourself. Where does that function come from?

it comes from the C standard library, which supplies reusable functions for common tasks such as input/output, string manipulation, mathematics, and file access.

#### What does #include <stdio.h> actually do?
you already learned that a header file provides declaration and related information needed by the compiler.

for this chapter, stdio.h declares standard I/O facilities, including functions such as:

- putchar() - output one character.
- getchar() - input one character.
- printf() - formatted output.
- scanf() - formatted input.
- fprintf() and fscanf() - formatted I/O using a specified stream, commonly a file.

The header generally does not contain the complete machine-code implementation of these library functions. The linker and runtime environment make the necessary implementation available to your program. depending on the platform and toolchain, that may involve static or dynamic linking.

![alt text](image-1.png)

Mental model: a header tells your source code how to call a function; it is not, by itself, the functions complete implementation.

This is the same distinction you learned earlier between declaring something and providing its implementation.