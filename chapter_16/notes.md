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