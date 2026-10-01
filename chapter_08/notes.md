The main point of this chapter is:

    How do we organize larger, more complicated information and larger programs?


```
                 CHAPTER 8
                     |
        +------------+------------+
        |                         |
   SUBROUTINES               DATA STRUCTURES
        |                         |
   call / return         +--------+--------+---------+
        |                |        |        |         |
   save registers      STACK    QUEUE   STRING   ...
        |
     enables
        |
    RECURSION
        |
      uses
      STACK
```

## What is a data structure?
Suppose i give you these values:
```
10
25
42
7
91
```

Those are just individual values.

But real programs often need to represent something more organized:
```
Employee
 ├── name
 ├── salary
 ├── age
 └── department
```

or:
```
People waiting in line
A → B → C → D
```

or:
```
Browser history
Page1 → Page2 → Page3
```

or:
```
Characters of a name
B i l l   L i n v i l l
```

the problem is no longer simply:

    How do i store a value?

it becomes:

    How should multiple values be organized, and what operations should be allowed on them?

That is where data structure enter.


### Abstract Data type

    An abstract data type is defined by what you can do with it, not by how it is physically implemented.

Let's take a stack

A stack says:
```
Last thing inserted
        ↓
    removed first
```

That rule is called:
#### LIFO - Last In, First Out

Suppose we do:
```
PUSH A
PUSH B
PUSH C
```

The stack is conceptually:
```
TOP
 ↓
 C
 B
 A
```

Then:
```
POP
```

gives:
```
C
```

Another:
```
POP
```

gives:
```
B
```

Important part is that the stack doesn't care whether it is implemented using:
```
memory
registers
an array
some hardware structure
```

The behavior is what defines the stack.

that is the meaning of the abstraction.


### Subroutines:
you have already seen assembly instruction like:
```
ADD
LDR
BR
STR
```
But imagine your program contains the same 20 instructions in five different places.

That would be ugly:
```
code
code
code
code
code

same code again

code

same code again

code
```

instead, we can write it once:
```
SUBROUTINE_A
    instructions...
    instructions...
    instructions...
    return
```

and jump to it whenever we need it.

this is the basic idea of a subroutine.

in C, the equivalent concept is a function.

for example:
```
void printHello(void)
{
    printf("Hello\n");
}
```

then:
```
printHello();
printHello();
printHello();
```

the code for the operation exists only once
