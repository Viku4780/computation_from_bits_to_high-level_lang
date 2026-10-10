## Understand what an array really is

### Why do we need arrays?
Imagine you want to store the marks of five students.

you could write:
```
int mark1 = 75;
int mark2 = 82;
int mark3 = 91;
int mark4 = 68;
int mark5 = 88;
```

This works, but suppose you have 100 students. you would need 100 separate variable names, and writing a loop to process those values would be awkward.

Instead, C lets you create an array:
```
int marks[5] = {75, 82, 91, 68, 88};
```

Read this declaration carefully:
- int means each element stores an integer.
- marks is the array's name.
- [5] means the array contains five elements.
- {75,82,91,68,88} supplies the initial values.

conceptually, we have created five integer elements grouped into one array.

![alt text](image.png)

The key benefit is that we can use a number called an index to select an element.

```
printf("%d\n", marks[0]);   // 75
printf("%d\n", marks[2]);   // 91
printf("%d\n", marks[4]);   // 88
```

### Why does indexing start at zero?
For an array containing five elements, the valid indices are:

![alt text](image-2.png)

The last index is not 5. it is 4.

the general rule is:

    last valid index = array length - 1

for an array of length n, valid indices are from 0 through n - 1.

Why zero? Because indexing can be understood as an offset from the beginning of an array.

- marks[0]: move zero elements from the beginning.
- marks[1]: move one element from the beginning.
- marks[2]: move two elements from the beginning.


### What happens in memory?
Suppose, just for illustration, that marks begins at memory address 1000, and each int occupies four bytes on this particular machine.

![alt text](image-3.png)

Three important observations:

1. The elements are stored in contiguous memory: one immediately follows another.
2. Each element occupies sizeof(int) bytes.
3. The address of an element can be calculated from the starting address and its index.

For an array of integers, the conceptual address calculation is:

    address of element i = base address + i * sizeof(int)

for example, if the base address is 1000 and sizeof(int) is 4, the address of marks[3] is:

    1000 + 3 * 4 = 1012

this calculation is the bridge between arrays and pointers.

One precision point: C guarantees that array elements are contiguous and that sizeof(marks) gives the total size of this complete array object. But C does not guarantee that an int is four bytes or that the array begins at any particular address. Those details depend on the implementation and the actual program execution.


### accessing and modifying elements
```
int marks[5] = {75, 82, 91, 68, 88};

marks[2] = 95;
marks[0] = 70;
```

After these assignments, the values are:
```
70, 82, 95, 68, 88
```

The array has the same number of elements. We have changed the values stored in two of those elements.

```
#include <stdio.h>

int main(void)
{
    int marks[5] = {75, 82, 91, 68, 88};

    for (int i = 0; i < 5; i++) {
        printf("%d\n", marks[i]);
    }

    return 0;
}
```

![alt text](image-4.png)

when i becomes 5, the condition i < 5 is false, so the loop stops.

Notice how the array and loop complement each other: the array gives the elements a predictable organization, while the loop systematically visits each index.


### A serious mistake: accessing beyond the array
Consider this:
```
int marks[5] = {75, 82, 91, 68, 88};

printf("%d\n", marks[5]);  // Wrong
```

Why is this wrong?

Because the valid indices are 0, 1, 2, 3, and 4. Index 5 refers to a position outside the array.

In C, accessing an array outside its valid bounds this way causes undefined behavior. It does not reliably produce a particular value, and the program might appear to work, print an unexpected value, corrupt other data, or crash.

A dangerous example is:
```
int marks[5];
int limit = 10;

for (int i = 0; i < limit; i++) {
    marks[i] = 0;
}
```
The loop attempts to write to ten elements even though only five exist. The error is in the mismatch between the array's capacity and the loop's limit.

The lesson: every array access must respect the array's actual bounds. C generally does not automatically check them for you.


## Pointers: variables that store addresses

### start with an ordinary variable
```
int number = 42;
```

Conceptually, memory contains a variable named number whose value is 42.

Now suppose its address is 2000 on our illustrative machine.

![alt text](image-5.png)

The variable's value is 42. its address is 2000. These are different things

C provides the address-of  operator &:
```
printf("%p\n", (void *)&number);
```

Here, &number means "the address of number." the cast to void * is the appropriate form for printing an object pointer with %p.


### What is a pointer?
A pointer is an object whose value is an address that can designate another object or function, depending on the pointer type.

For example:
```
int number = 42;
int *ptr = &number;
```

Let's read the second line from left to right.

- int *ptr declares ptr as a pointer to an int.
- &number obtains the address of number.
- = initializes ptr with that address.

If number is at address 2000, then ptr stores that address.

![alt text](image-6.png)

A pointer is not the value 42 in this example. It stores the address of the object that contains 42.


### What does *ptr mean?
We have now seen two operators:

- &number: obtain the address of number.
- *ptr: dereference ptr, accessing the object to which it points.

For example:
```
int number = 42;
int *ptr = &number;

printf("%d\n", *ptr);  // 42
```
The expression *ptr means, in this context, “the int object reached through the address stored in ptr.”

Now consider:
```
*ptr = 99;
```

This does not replace the pointer's stored address. It changes the value of the object to which the pointer points.

After that assignment:
```
printf("%d\n", number);  // 99
```
Why does number now contain 99? Because ptr points to number. The expression *ptr accesses the same object as the name number.

Here is the essential relationship:

![alt text](image-7.png)


### Do not confuse the two uses of *
Look at these lines:
```
int *ptr = &number;
*ptr = 99;
```
The symbol * has two related but distinct uses.

- In a declaration, int *ptr says that ptr is a pointer to int.
- In an expression, *ptr dereferences the pointer.

That difference depends on the context in which the symbol appears.

Also remember: a pointer must contain a valid address before you dereference it. For example:
```
int *ptr;
*ptr = 99;   // Wrong: ptr was not initialized
```
Here, ptr has an indeterminate value. Dereferencing it is not valid. A pointer is not automatically initialized to point to a useful object.


## the connection between pointer and arrays

### The address of the first array element
Consider:
```
int numbers[4] = {10, 20, 30, 40};
```
We know the array contains four integers in contiguous memory.

Now look at these two expressions:
```
numbers
&numbers[0]
```

In most expressions, the array name numbers is automatically converted into a pointer to its first element. Therefore, in this context, numbers and &numbers[0] point to the same first element.

For example:
```
int *ptr = numbers;
```
This is valid C. It initializes ptr to point to numbers[0].

Compare it with:
```
int *ptr = &numbers[0];
```
These initializations have the same effect.

![alt text](image-8.png)

The array is still an array. The pointer is still a pointer. They are related because the array name can convert to a pointer to its first element in most expressions.

That last qualification matters: an array is not itself a pointer. We will see why shortly.


### Pointer arithmetic: moving from one element to another
Suppose:
```
int numbers[4] = {10, 20, 30, 40};
int *ptr = numbers;
```
Initially, ptr points to numbers[0].

What does this do?
```
ptr++;
```
It advances the pointer to the next int element.

It does not necessarily add one byte to the address. C pointer arithmetic scales the movement by the size of the pointed-to type.

If sizeof(int) == 4 and the first element is at illustrative address 1000, the progression looks like this:

![alt text](image-9.png)

For a pointer ptr to an element of an array, these operations are meaningful:
```
ptr++;      // Advance by one int element
ptr--;      // Move back by one int element
ptr + 2;    // Pointer two int elements forward
ptr - 1;    // Pointer one int element backward
```

Pointer arithmetic is defined within an array object and permits forming a pointer one element past the end. That one-past-the-end pointer may be used for comparisons and loop termination, but must not be dereferenced.

You must not move a pointer arbitrarily outside the array's allowed range and then dereference it.