  
    An ordinary variable stores a value. A data structure gives us a way to organize related information. A dynamic data structure lets that organization grow or shrink while the program runs.

## The problem we trying to solve
Imagine that you are writing an application that tracks aircraft flying over a particular region. For every aircraft, you need to store:

- Its flight ID, such as ZA123
- Its altitude
- Its latitude and longitude
- Its heading
- Its airspeed

One aircraft is not represented by one simple number. It is a collection of different kinds of information that all describe the same real-world object. The textbook uses this aircraft-tracking example to introduce structures and then develops it into a dynamic linked list.

### Why not use separate variables?
We could write:
```
char flightID[7];
int altitude;
int longitude;
int latitude;
int heading;
double airSpeed;
```

This works for one aircraft. But what happens when we need to track 100 aircraft?

We might create separate arrays:
```
char flightIDs[100][7];
int altitudes[100];
int longitudes[100];
int latitudes[100];
int headings[100];
double airSpeeds[100];
```

Now we must remember that flightIDs[5], altitudes[5], latitudes[5], and airSpeeds[5] all describe the same aircraft.

That is possible, but the program becomes harder to understand and maintain.
Accidently using index 5 for one property and index 6 for another could mix up two aircraft.

The first lesson: when several values belong together conceptually, it is useful to group them together in one data object.

C provides a mechanism for doing exactly that: the struct.


## Structures in C: creating your own data type

### What is a struct?
Think of a structure as a form with several labeled fields.

An employee form might have a name, an ID, a salary, and a department. An aircraft form might have a flight ID, altitude, and airspeed.

A C structure lets us define that form once and then create as many individual objects of that form as we need.

Here is the textbook's aircraft structure, simplified slightly for learning:

```
struct Flight {
    char ID[7];
    int altitude;
    int longitude;
    int latitude;
    int heading;
    double airSpeed;
};
```

Let's understand every part
- struct tells C that we are declaring a structure type.
- Flight is the structure's tag, which identifies the type.
- char ID[7] is an array capable of holding up to six characters plus the terminationg '\0' character.
- The int members hold interger-valued properties.
- double airspeed holds the airspeed value.
- the semicolon after the closing brace is required.

Notice that we have described a type, not yet created an actual aircraft variable.


### Type description versus actual object
compare these two lines:
```
struct Flight {
    int altitude;
    double airSpeed;
};

struct Flight plane;
```

The first tells the compiler what a struct Flight contains.

the second creates a real variable called plane that has that structure type.

A useful analogy is a blueprint and a building:
- the structure declaration is the blueprint.
- plane is an actual object created using blueprint.

The blueprint itself is not the aircraft data. The variable is where the actual member values are stored.

### Accessing structure members with .
Once we have a structure variable, we access an individual member using the dot operator.

```
struct Flight plane;

plane.altitude = 10000;
plane.airSpeed = 800.0;
```

Read the first assignment as:

    Access the altitude member belonging to the structure variable plane, then store 10000 in that member.

The dot does not mean multiplication, and it does not access a separate global variable. it selects a field inside a particular structure object.

For example:
```
struct Flight plane1;
struct Flight plane2;

plane1.altitude = 10000;
plane2.altitude = 12000;
```

Both objects have a member called altitude, but they store separate values

![alt text](image.png)


### How does a structure occupy memory?
A structure's members occupy a region of memory allocated for that structure object. The compiler knows the member layout and generates instructions to access the appropriate member.

Conceptually, the memory could look like this:

![alt text](image-1.png)

This is a conceptual diagram, not a guarantee of exact byte offsets. Real C implementations may insert padding bytes between members to satisfy alignment requirements. Therefore, do not assume a structure's size is always the simple sum of the sizes you expect.

To find the actual size on the system compiling your program, use:
```
printf("%zu\n", sizeof(struct Flight));
```

The sizeof operator returns the size in bytes of the type or object on that implementation. The textbook highlights this portability issue when discussing memory allocation.

### typedef: giving a type a shorter name
you may have seen this:
```
typedef struct Flight Flight;
```

The purpose of typedef is to create a type alias. it does not create a new object or change the underlying structure layout.

After that declaration, you can write:
```
Flight plane;
```

instead of:
```
struct Flight plane;
```

The textbook uses this technique to make its aircraft type easier to use throughout the program.

here is the complete example:
```
#include <stdio.h>

typedef struct {
    int id;
    double salary;
} Employee;

int main(void)
{
    Employee worker;

    worker.id = 101;
    worker.salary = 45000.0;

    printf("ID: %d\n", worker.id);
    printf("Salary: %.2f\n", worker.salary);

    return 0;
}
```
employee is now the type name we use to declare employee objects.

Checkpoints: a struct groups different members into one object; . selects a member; typedef gives a type a convenient alias.