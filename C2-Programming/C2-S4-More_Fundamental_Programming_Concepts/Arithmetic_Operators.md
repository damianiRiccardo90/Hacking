# *__Arithmetic Operators__*

The statement _b = a + 7_ is an example of a very simple arithmetic operator. In C, the following symbols are used for various arithmetic operations.

The first four operations should look familiar. Modulo reduction may seem like a new concept, but it's really just taking the remainder after division. If __a__ is 13, then 13 divided by 5 equals 2, with a remainder of 3, which means that _a % 5 = 3_. Also, since the variables __a__ and __b__ are integers, the statement _b = a / 5_ will result in the value of 2 being stored in __b__, since that's the integer portion of it. Floating-point variables must be used to retain the more correct answer of 2.6.

<div align="left" width="100%">
<img src="Operation_Examples.png?raw=true" alt="Operation Examples" width="50%">
</div>

To get a program to use these concepts, you must speak its language. The C language also provides several forms of shorthand for these arithmetic operations. One of these was mentioned earlier and is used commonly in for loops.

<div align="left" width="100%">
<img src="Shorthand_Operations.png?raw=true" alt="Shorthand Operations" width="50%">
</div>

These shorthand expressions can be combined with other arithmetic operations to produce more complex expressions. This is where the difference between _i++_ and _++i_ becomes apparent. The first expression means _Increment the value of __i__ by 1_ after _evaluating the arithmetic operation_, while the second expression means _Increment the value of __i__ by 1_ before _evaluating the arithmetic operation_. The following example will help clarify.

```c
int a, b;
a = 5;
b = a++ * 6;
```

At the end of this set of instructions, __b__ will contain 30 and __a__ will contain 6, since the shorthand of _b = a++ * 6;_ is equivalent to the following statements:

```c
b = a * 6;
a = a + 1;
```

However, if the instruction _b = ++a * 6;_ is used, the order of the addition to __a__ changes, resulting in the following equivalent instructions:

```c
a = a + 1;
b = a * 6;
```

Since the order has changed, in this case __b__ will contain 36, and __a__ will still contain 6.

Quite often in programs, variables need to be modified in place. For example, you might need to add an arbitrary value like 12 to a variable, and store the result right back in that variable (for example, _i = i + 12_). This happens commonly enough that shorthand also exists for it.

<div align="left" width="100%">
<img src="More_Shorthand_Operations.png?raw=true" alt="More Shorthand Operations" width="70%">
</div>