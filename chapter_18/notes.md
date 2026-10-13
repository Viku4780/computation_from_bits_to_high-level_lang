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


#### 4.3 why does getchar() sometimes appear to wait for Enter?
try this program:
```
#include <stdio.h>

int main(void)
{
    int first;
    int second;

    printf("Input character 1:\n");
    first = getchar();

    printf("Input character 2:\n");
    second = getchar();

    printf("First: %c\n", first);
    printf("Second: %c\n", second);

    return 0;
}
```

Suppose you type A and press Enter.

on a typical terminal with line-buffered keyboard input, the input stream can receive both the A and the newline character. The first getchar() reads A; the second can read the newline left behind by pressing Enter.

So the second call may not wait for another keypress at all.

![alt text](image-4.png)

This behavior is related to buffering.

The terminal or runtime commonly collects typed characters in an input buffer until a line is submitted. This lets a user correct a typo before the program consumes the line. Once the input is made available, character-reading functions consume characters from that stream in order.

A useful mental model is:

1. You type characters.
2. The terminal/input layer collects them.
3. Pressing Enter commonly makes the line available to the program.
4. each getchar() reads the next available character.
5. The newline may remain for the next read.

Do not assume that every environment uses exactly the same buffering behavior. Interactive terminals, redirected input, IDE consoles, and other environments can behave differently. The key idea is that the function reads from a stream; it does not necessarily communicate with a physical keypress directly.


#### 4.4 Output buffering: why output might appear later
Output can also be buffered. A program can write characters to an output stream, and the runtime may collect them before sending them to the destination. the timing depends on the stream and environment. The book illustrates this with output that appears only after a newline flushes the buffer.

For example:
```
printf("Working ...");
```
The absence of a newline means you should not assume that the text will immediately become visible in every environment.

if you need to request that buffered output be delivered, C provides:
```
fflush(stdout);
```

for example:
```
printf("working ...");
fflush(stdout);
```

This is especially useful for prompts that should appear before the program waits for input. Be careful: fflush(stdin) is not a portable way to discard keyboard input in standard C.

The distinction is between writing data to a stream and the data becoming visible at the final destination. Buffering helps the system operate efficiently, but it can make timing less obvious.


## Formatted output with printf()
Consider:
```
int age = 20;
printf("Age = %d\n", age);
```

The function receives two things:

1. The format string: "Age = %d\n"
2. The additional argument: age, whose value is 20

the %d tells printf() how to convert the corresponding argument into text. The \n represents a newline.

The output is:
```
Age = 20
```

the program stores age as an interger. The output stream, however, receives characters representing the number. That conversion is part of printf()'s job.


### 5.1 The format string is a set of instructions
Look at this example:
```
int age = 20;
float temperature = 26.5f;

printf("Age: %d, temperature: %.1f\n", age, temperature);
```

Read the format string from left to right:

- Age: is ordinary text.
- %d says to format the next argument as a decimal integer.
- , temperature: is ordinary text.
- %.1f says to format the next argument as a floating-point number with one digit after the decimal point.
- \n adds a newline.

the output is:
```
Age: 20, temperature: 26.5
```

The format string controls how the arguments are interpreted and displayed


### 5.2 Common conversion specifiers

![alt text](image-5.png)

The book also discusses converting integers into hexadecimal and character output, as well as outputting strings stored in consecutive memory locations.

A note about standards: the book mentions %b when discussing binary output, but %b is not a portable conversion specifier in C18, the C version identified by the book. Support depends on the C standard and compiler/library implementation. For code intended to work broadly with C18, do not assume that printf("%b", value) will print binary.


### 5.3 Formatting width and precision
Conversion specifiers can contain extra formatting instructions.

```
int number = 42;
double price = 7.5;

printf("%5d\n", number);
printf("%.2f\n", price);
printf("%8.2f\n", price);
```

Conceptually:

- %5d uses a minimum field width of five characters.
- %.2f prints two digits after the decimal point.
- %8.2f combines a minimum width of eight characters with two digits after the decimal point.

Typical output:
```
   42
7.50
    7.50
```

width does not automatically truncate a value that is too large for the field. It specifies a minimum width.


### 5.4 The dangerous mistake: the format and argument disagree
Consider:
```
float x = 192.27163f;

printf("%d\n", x);   // Incorrect
```

This is not a valid way to print a floating-point value. %d tells printf() to retrieve an int argument, but the expression x is passed as a floating-point argument. The format and argument types do not match, which causes undefined behavior.

The correct call is:
```
printf("%f\n", (double)x);
```

In a variadic function such as printf(), a float argument is promoted to double, so you could also write:
```
printf("%f\n", x);
```

The cast is not required here.

This connects to the memory and type model you learned earlier: the same sequence of bits cannot safely be interpreted as any type we happen to request. The function call has to obey the type requirements associated with its format string.

Rule: every conversion specifier must match the type of the corresponding argument, subject to the standard's specified argument promotions.


## scanf(): formatted input is not the reverse of printing
it is tempting to think that scanf() simply reverses printf(). the two functions are related, but input introduces a separate problem: the characters in the stream may not represent the value your program expects.

Consider:
```
int age;

scanf("%d", &age);
```

The %d tells scanf() to read a sequence of characters representing a decimal integer convert that sequence into an integer value, and store it at the location supplied by &age.


### 6.1 Why does scanf() need &age?
You learned about pointers in Chapter 16.

Let's recall:
```
int age;
```
age is an integer object. Its value is not the same thing as its address.

When we write:
```
scanf("%d", &age);
```
we pass the address of age to scanf(). That gives the function a location where it can store the converted value.

If you instead wrote:

scanf("%d", age);  // Incorrect

you would pass the integer value of age where scanf() expects a pointer to an integer. This is a type mismatch and can cause undefined behavior.

The memory-level sequence is:

![alt text](image-6.png)

The & is not a special scanf() operator. It is the ordinary C address-of operator you already know. scanf() needs the address because its job includes writing into your variable.

One exception worth remembering: when you pass a character array as a %s destination, the array expression normally converts to a pointer to its first element, so you write scanf("%s", name), not scanf("%s", &name).

### 6.2 scanf() with multiple values

```
int day, month, year;

scanf("%d/%d/%d", &day, &month, &year);
```

if the input is:
```
12/10/2026
```

the conversion specifications read the three decimal numbers. The literal / characters in the format string tell scanf() to match slash characters in the input stream.

This is an important point: characters in a scanf() format string are not all conversion instructions. Some are literal characters that the input must match.

The book demonstrates this behavior with a format string containing %d/%d/%d and explains how the numbers are assigned to the locations provided as arguments.


### 6.3 Whitespace: why spaces and newlines can behave unexpectedly

Most whitespace in a scanf() format string—including spaces, tabs, and newlines—directs the function to skip any amount of input whitespace. Many conversions, including %d and %s, also skip leading whitespace automatically.

for example:
```
int x;
scanf("%d", &x);
```
if the stream begins with spaces followed by 123, the leading whitespace is skipped and x becomes 123.

But %c behaves differently:
```
char ch;
scanf("%c", )