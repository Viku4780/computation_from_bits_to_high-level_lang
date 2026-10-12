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


### I/O streams: think of data as a flow
the book's central abstraction is the stream. Rather than forcing a program to communicate directly with the physical keyboard or display every time, the I/O system presents a sequence of data that can be read or written.

imagine  a stream as a queue of characters:

![alt text](image-2.png)

The producer put data into a stream; the consumer reads it. The two sides can work at different rates, which is one reason the abstraction is useful.

for example, a user may type several characters faster than you program processes them likewise, your program may generate output faster than a display can show it.

#### The three standard streams
C provides three familiar standard streams

![alt text](image-3.png)

the book focuses primaarily on stdin and stdout, then extends the same stream idea to files. stderr is also part of standard C and is useful to know in practical programs.

Here is a small example:
```
#include <stdio.h>

int main(void)
{
    printf("Normal output\n);
    fprintf(stderr, "Diagnostic message\n");

    return 0;
}
```

printf() writes to stdout. fprintf(stderr, ...) writes to stderr.

the terminal may display both messages together, but they are logically separate streams. this becomes useful when output is redirected to a file while errors remain visible in the terminal.

### Character I/O: putchar() and getchar()

#### 4.1 putchar() - output one character
```
putchar('A');
```
This sends the character A to standard output.

Compara these cells:
```
putchar('A');
putchar(65);
putchar('B');
```

on an ASCII-compatible system, the first two calls output the same character because the ASCII code for A is 65. the third outputs B.

The important distinction is that putchar() does not  format an integer as a decimal number. it treats its argument as a character value and writes that character to the output stream.

compare:
```
printf("%d\n", 65);
putchar(65);
```

The first outputs the two characters 6 and 5, followed by a newline. The second outputs the single character A on an ASCII-compatible system.

Why? Because the two functions have different jobs:

- printf(%d", 65) converts the number 65 into the textual representation "65".
- putchar(65) writes the character represented by character code 65.

this is the difference between formatting a numeric value and outputting a character.

#### 4.2 getchar() - read one character
```
int ch = getchar();
```

getchar() reads the next character from stdin and returns its value. it returns an int, not a char, because it must also be able to represent the special value EOF(end-of-file or input failure indication).

For a beginner, the important lesson is to store its result in an int when you need to detect EOF reliably.
```
#include <stdio.h>

int main(void)
{
    int ch;

    ch = getchar();

    if(ch != EOF){
        putchar(ch);
        putchar('\n');
    }

    return  0;
}
```

The program reads one charater and writes it back, provided the read did not return EOF.

Connect to LC-3: the book relates getchar() and putchar() to the LC-3 IN and OUT trap routines. In both cases, the program uses a higher-level interface to read or output character data; the C library hides the lower-level mechanics