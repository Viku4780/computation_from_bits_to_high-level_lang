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