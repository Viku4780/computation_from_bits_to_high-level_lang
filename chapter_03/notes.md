## What is Transistor?
an electrically controlled switch

the transistor has three terminals:

```
        Gate
          │
          │
       ┌─────┐
       │     │
Source │     │ Drain
       └─────┘
```

The important terminal for us is the :
Gate

because the voltage applied to the gate controls whether the path between source and drain is effectively connected.


## N-type transistor

The book introduces two types:

N-type
P-type


The simplified behavior is:

N-type:

Gate = 1
   ↓
Source-Drain path CLOSED

Gate = 0
   ↓
Source-Drain path OPEN

```
N-type

       gate
         │
         ▼
      ┌──────┐
  ────┤      ├────
      └──────┘

1 → switch ON
0 → switch OFF
```

## P-type transistor
behaves opposite

```
P-type:

Gate = 0
   ↓
Source-Drain path CLOSED

Gate = 1
   ↓
Source-Drain path OPEN
```


## Why do we have both N and P?

Because together they allow us to build extremely useful circuits where:
```
one path pulls output HIGH
another path pulls output LOW
```

And they can be arranged so that the output always gets a strong, well-defined logical value.

Circuits built using complementary P-type and N-type transistors are called CMOS circuits.

CMOS means:
Complementary Metal-Oxide Semiconductor.

```
CMOS
=
P-type + N-type
```