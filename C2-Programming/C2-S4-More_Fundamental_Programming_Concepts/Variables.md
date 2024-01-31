# *__Variables__*

The counter used in the for loop is actually a type of variable. A __variable__ can simply be thought of as an object that holds data that can be changed, hence the name. There are also variables that don't change, which are aptly called __constants__. Returning to the driving example, the speed of the car would be a variable, while the color of the car would be a constant. In pseudo-code, variables are simple abstract concepts, but in C (and in many other languages), variables must be declared and given a type before they can be used. This is because a C program will eventually be compiled into an executable program. Like a cooking recipe that lists all the required ingredients before giving the instructions, variable declarations allow you to make preparations before getting into the meat of the program. Ultimately, all variable are stored in memory somewhere, and their declarations allow the compiler to organize this memory more efficiently. In the end though, despite all of the variable type declarations, everything is all just memory.

In C, each variable is given a type that describes the information that is meant to be stored in that variable. Some of the most common types are __int__ (integer values), __float__ (decimal floating-point values), and __char__ (single character values). Variables are declared simply by using these keywords before listing the variables, as you can see below.

```c
int a, b;
float k;
char z;
```

The variables __a__ and __b__ are now defined as integers, __k__ can accept floating-point values (such as 3.14), and __z__ is expected to hold a character value, like _A_ or _w_. Variables can be assigned values when they are declared or anytime afterward, using the __=__ operator.

```c
int a = 13, b;
float k;
char z = 'A';

k = 3.14;
z = 'w';
b = a + 5;
```

After the following instructions are executed, the variable __a__ will contain the value of 13, __k__ will contain the number 3.14, __z__ will contain the character _w_, and __b__ will contain the value 18, since 13 plus 5 equals 18. Variables are simply a way to remember values; however, with C, you must first declare each variable's type.