# *__Signed, Unsigned, Long, and Short__*

By default, numerical values in C are signed, which means they can be both negative and positive. In contrast, unsigned values don't allow negative numbers. Since it's all just memory in the end, all numerical values must be stored in binary, and unsigned values make the most sense in binary. A 32-bit unsigned integer can contain values from 0 (all binary 0s) to _4,294,967,295_ (all binary 1s). A 32-bit singed integer is still just 32 bits, which means it can only be in one of $ 2^{32} $ possible bit combinations. This allows 32-bit signed integers to range from _-2,147,483,648_ to _2,147,483,647_. Essentially, one of the bits is a flag marking the value positive or negative. Positively signed values look the same as unsigned values, but negative numbers are store differently using a method called two's complement. _Two's complement_ represents negative numbers in a form suited for binary adders, when a  value in two's complement is added to a positive number of the same magnitude, the result will be 0. This is done by first writing the positive number in binary, then inverting all the bits, and finally adding 1. It sounds strange, but it works and allows negative numbers to be added in combination with positive numbers using simple binary adders.

This can be explored quickly on a smaller scale using __pcalc__, a simple programmer's calculator that displays results in decimal, hexadecimal, and binary formats. For simplicity's sake, 8-bit numbers are used in this example.

<pre style="color: white;">
reader@hacking:~/booksrc $ pcalc 0y01001001
73 0x49 0y1001001
reader@hacking:~/booksrc $ pcalc 0y10110110 + 1
183 0xb7 0y10110111
reader@hacking:~/booksrc $ pcalc 0y01001001 + 0y10110111
256 0x100 0y100000000
reader@hacking:~/booksrc $
</pre>

First, the binary value _01001001_ is shown to be positive 73. Then all the bits are flipped, and 1 is added to result in the two's complement representation for negative 73, _10110111_. When these two values are added together, the result of the original 8 bits is 0. The program _pcalc_ shows the value 256 because it's not aware that we're only dealing with 8-bit values. In a binary adder, that carry bit would just be thrown away because the end of the variable's memory would have been reached. This example might shed some light on how two's complement works its magic.

In C, variables can be declared as unsigned by simply prepending the keyword _unsigned_ to the declaration. An unsigned integer would be declared with _unsigned int_. In addition, the size of numerical variables can be extended or shortened by adding the keywords __long__ or __short__. The actual sizes will vary depending on the architecture the code is compiled for. The language of C provides a macro called __sizeof()__ that can determine the size of certain data type. This works like a function that takes a data type as its input and returns the size of a variable declared with that dat type for the target architecture. The _datatype_sizes.c_ program explores the sizes of various data types, using the _sizeof()_ function.

__datatype_sizes.c__

```c
#include <stdio.h>

int main()
{
    printf("The 'int' data type is \t\t %d bytes\n", sizeof(int));
    printf("The 'unsigned int' data type is\t %d bytes\n", sizeof(unsigned int));
    printf("The 'short int' data type is\t %d bytes\n", sizeof(short int));
    printf("The 'long int' data type is\t %d bytes\n", sizeof(long int));
    printf("The 'long long int' data type is %d bytes\n", sizeof(long long int));
    printf("The 'float' data type is\t %d bytes\n", sizeof(float));
    printf("The 'char' data type is\t\t %d bytes\n", sizeof(char));
}
```

This piece of code uses the _printf()_ function in a slightly different way. It uses something called a format specifier to display the value returned from the _sizeof()_ function calls. Format specifiers will be explained in depth later, so for now, let's just focus on the program's output.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc datatype_sizes.c
reader@hacking:~/booksrc $ ./a.out
The 'int' data type is 4 bytes
The 'unsigned int' data type is 4 bytes
The 'short int' data type is 2 bytes
The 'long int' data type is 4 bytes
The 'long long int' data type is 8 bytes
The 'float' data type is 4 bytes
The 'char' data type is 1 bytes
reader@hacking:~/booksrc $
</pre>

As previously stated, both signed and unsigned integers are four bites in size on the x86 architecture. A float is also four bytes, while a char only needs a single byte. The _long_ and _short_ keywords can also be used with floating-point variables to extend and shorten their sizes.