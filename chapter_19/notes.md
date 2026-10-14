  
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

