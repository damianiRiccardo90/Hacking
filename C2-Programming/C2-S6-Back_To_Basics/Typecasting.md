# *__Typecasting__*

_Typecasting_ is simply a way to temporarily change a variables' data type, despite how iut was originally defined. When a variable is typecast into a different type, the compiler is basically told to treat that variable as if it were the new data type, but only for that operation. The syntax for typecasting is as follows:

<pre style="color: white;">
(typecast_data_type) variable
</pre>

This can be done when dealing with integers and floating-point variables, as _typecasting.c_ demonstrates.

__typecasting.c__

```c
#include <stdio.h>

int main()
{
    int a, b;
    float c, d;

    a = 13;
    b = 5;

    c = a / b;                 // Divide using integers.
    d = (float) a / (float) b; // Divide integers typecast as floats.

    printf("[integers]\t a = %d\t b = %d\n", a, b);
    printf("[floats]\t c = %f\t d = %f\n", c, d);
}
```

The results of compiling and executing _typecasting.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc typecasting.c
reader@hacking:~/booksrc $ ./a.out
[integers] a = 13 b = 5
[floats] c = 2.000000 d = 2.600000
reader@hacking:~/booksrc $
</pre>

As discussed earlier, diving the integer _13_ by _5_ will round down to the incorrect answer of _2_, even if this value is being stored into a floating-point variable. However, if these integer variables are typecast into floats, they will be treated as such. This allows for the correct calculation of _2.6_.

This example is illustrative, but where typecasting really shines iw when it is used with pointer variables. Even though a pointer is just a memory address, the C compiler still demands a data type for every pointer. One reason for this is to try to limit programming errors. An integer pointer should only point to integer data, while a character pointer should only point to character data. Another reason is for pointer arithmetic. An integer is four bytes in size, while a character only takes up a single byte. The _pointer_types.c_ program will demonstrate and explain these concepts further. This code uses the format parameter __%p__ to output memory addresses. This is shorthand meant for displaying pointers and is basically equivalent to _0x%08x_.

__pointer_types.c__

```c
#include <stdio.h>

int main() 
{
    int i;

    char char_array[5] = {'a', 'b', 'c', 'd', 'e'};
    int int_array[5] = {1, 2, 3, 4, 5};

    char* char_pointer;
    int* int_pointer;

    char_pointer = char_array;
    int_pointer = int_array;

    // Iterate through the int array with the int_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[integer pointer] points to %p, which contains the integer %d\n",
            int_pointer, *int_pointer);
        int_pointer = int_pointer + 1;
    }

    // Iterate through the char array with the char_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[char pointer] points to %p, which contains the char '%c'\n",
            char_pointer, *char_pointer);
        char_pointer = char_pointer + 1;
    }
}
```

In this code two arrays are defined in memory, one containing integer data and the other containing character data. Two pointers are also defined, one with the integer data type and one with the character data type, and they are set to point at the start of the corresponding data arrays. Two separate for loops iterate through the arrays using pointer arithmetic to adjust the pointer to point at the next value. In the loops, when the integer and character values are actually printed with the __%d__ and __%d__ format parameters, notice that the corresponding _printf()_ arguments must dereference the pointer variables. This is done using the _unary * operator_.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc pointer_types.c
reader@hacking:~/booksrc $ ./a.out
[integer pointer] points to 0xbffff7f0, which contains the integer 1
[integer pointer] points to 0xbffff7f4, which contains the integer 2
[integer pointer] points to 0xbffff7f8, which contains the integer 3
[integer pointer] points to 0xbffff7fc, which contains the integer 4
[integer pointer] points to 0xbffff800, which contains the integer 5
[char pointer] points to 0xbffff810, which contains the char 'a'
[char pointer] points to 0xbffff811, which contains the char 'b'
[char pointer] points to 0xbffff812, which contains the char 'c'
[char pointer] points to 0xbffff813, which contains the char 'd'
[char pointer] points to 0xbffff814, which contains the char 'e'
reader@hacking:~/booksrc $
</pre>

Even though the same value of 1 is added to __int_pointer__ and __char_pointer__ in their respective loops, the compiler increments the pointer's addresses by different amounts. Since a char is only 1 byte, the pointer to the next char would naturally also by 1 byte over. But since an integer is 4 bytes, a pointer to the next integer has to be 4 bytes over.

In _pointer_types2.c_, the pointers are juxtaposed such that the __int_pointer__ points to the character data and vice versa.

__pointer_types2.c__

```c
#include <stdio.h>

int main() 
{
    int i;
    char char_array[5] = {'a', 'b', 'c', 'd', 'e'};
    int int_array[5] = {1, 2, 3, 4, 5};

    char* char_pointer;
    int* int_pointer;

    char_pointer = int_array; // The char_pointer and int_pointer now
    int_pointer = char_array; // point to incompatible data types.

    // Iterate through the int array with the int_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[integer pointer] points to %p, which contains the char '%c'\n",
            int_pointer, *int_pointer);
        int_pointer = int_pointer + 1;
    }

    // Iterate through the char array with the char_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[char pointer] points to %p, which contains the integer %d\n",
            char_pointer, *char_pointer);
        char_pointer = char_pointer + 1;
    }
}
```

The output below shows the warning spewed forth from the compiler.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc pointer_types2.c
pointer_types2.c: In function `main':
pointer_types2.c:12: warning: assignment from incompatible pointer type
pointer_types2.c:13: warning: assignment from incompatible pointer type
reader@hacking:~/booksrc $
</pre>

In an attempt to prevent programming mistakes, the compiler gives warnings about pointers that point to incompatible data types. But the compiler and perhaps the programmer are the only ones that care about a pointer's type. In the compiled code, a pointer is nothing more than a memory address, so the compiler will still compile the code if a pointer points to an incompatible data type, it simply warns the programmer to anticipate unexpected results.

<pre style="color: white;">
reader@hacking:~/booksrc $ ./a.out
[integer pointer] points to 0xbffff810, which contains the char 'a'
[integer pointer] points to 0xbffff814, which contains the char 'e'
[integer pointer] points to 0xbffff818, which contains the char '8'
[integer pointer] points to 0xbffff81c, which contains the char '
[integer pointer] points to 0xbffff820, which contains the char '?'
[char pointer] points to 0xbffff7f0, which contains the integer 1
[char pointer] points to 0xbffff7f1, which contains the integer 0
[char pointer] points to 0xbffff7f2, which contains the integer 0
[char pointer] points to 0xbffff7f3, which contains the integer 0
[char pointer] points to 0xbffff7f4, which contains the integer 2
reader@hacking:~/booksrc $
</pre>

Even though the __int_pointer__ points to character data that only contains 5 bytes of data, it is still typed as an integer. This means that adding 1 to the pointer will increment the address by 4 each time. Similarly, the __char_pointer__'s address is only incremented by 1 each time, stepping through the 20 bytes of integer data (five 4-byte integers), one byte at a time. Once again, the little-endian byte order of the integer data is apparent when the 4-byte integer is examined one byte at a time. The 4-byte value of _0x00000001_ is actually stored in memory as _0x01_, _0x00_ _0x00_, _0x00_.

There will be situations like this in which you are using a pointer that points to data with a conflicting type. Since the pointer type determines the size of the data it points to, it's important that the type is correct. As you can see in _pointer_types3.c_ below, typecasting is just a way to change the type of a variable on the fly.

__pointer_types3.c__

```c
#include <stdio.h>

int main() 
{
    int i;
    char char_array[5] = {'a', 'b', 'c', 'd', 'e'};
    int int_array[5] = {1, 2, 3, 4, 5};

    char* char_pointer;
    int* int_pointer;

    char_pointer = (char*) int_array; // Typecast into the
    int_pointer = (int*) char_array;  // pointer's data type.

    // Iterate through the int array with the int_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[integer pointer] points to %p, which contains the char '%c'\n",
            int_pointer, *int_pointer);
        int_pointer = (int*) ((char*) int_pointer + 1);
    }

    // Iterate through the char array with the char_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[char pointer] points to %p, which contains the integer %d\n",
            char_pointer, *char_pointer);
        char_pointer = (char*) ((int*) char_pointer + 1);
    }
}
```

In this code, when the pointers are initially set, the data is typecast into the pointer's data type. This will prevent the C compiler from complaining about the conflicting data types; however, any pointer arithmetic will still be incorrect. To fix that, when 1 is added to the pointers, they must first be typecast into the correct data type so the address is incremented by the correct amount. Then this pointer needs to be typecast back into the pointer's data type once again. It doesn't look too pretty, but it works.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc pointer_types3.c
reader@hacking:~/booksrc $ ./a.out
[integer pointer] points to 0xbffff810, which contains the char 'a'
[integer pointer] points to 0xbffff811, which contains the char 'b'
[integer pointer] points to 0xbffff812, which contains the char 'c'
[integer pointer] points to 0xbffff813, which contains the char 'd'
[integer pointer] points to 0xbffff814, which contains the char 'e'
[char pointer] points to 0xbffff7f0, which contains the integer 1
[char pointer] points to 0xbffff7f4, which contains the integer 2
[char pointer] points to 0xbffff7f8, which contains the integer 3
[char pointer] points to 0xbffff7fc, which contains the integer 4
[char pointer] points to 0xbffff800, which contains the integer 5
reader@hacking:~/booksrc $
</pre>

Naturally, it is far easier just to use the correct data type for pointers in the first place; however, sometimes a generic, typeless pointer is desired. In C, a void pointer is a typeless pointer, defined by the __void__ keyword.

Experimenting with void pointers quickly reveals a few things about typeless pointers. First, pointers cannot be dereferenced unless they have a type. In order to retrieve the value stored in the pointer's memory address, the compiler must first know what type of data it is. Secondly, void pointes must also be typecast before doing pointer arithmetic. These are fairly intuitive limitations, which means that a void pointer's main purpose is to simply hold a memory address.

The _pointer_types3.c_ program can be modified to use a single void pointer by typecasting it to the proper type each time it's used. The compiler knows that a void pointer is typeless, so any type of pointer can be stored in a void pointer without typecasting. This also means a void pointer must always be typecast when dereferencing it, however. These differences can be seen in _pointer_types4.c_, which uses a void pointer.

___pointer_types3.c__

```c
#include <stdio.h>

int main() 
{
    int i;

    char char_array[5] = {'a', 'b', 'c', 'd', 'e'};
    int int_array[5] = {1, 2, 3, 4, 5};

    void* void_pointer;

    void_pointer = (void*) char_array;

    // Iterate through the int array with the int_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[char pointer] points to %p, which contains the char '%c'\n",
            void_pointer, *((char*) void_pointer));
        void_pointer = (void*) ((char*) void_pointer + 1);
    }

    void_pointer = (void*) int_array;

    // Iterate through the int array with the int_pointer.
    for(i = 0; i < 5; i++) 
    {
        printf("[integer pointer] points to %p, which contains the integer %d\n",
            void_pointer, *((int*) void_pointer));
        void_pointer = (void*) ((int*) void_pointer + 1);
    }
}
```

The results of compiling and executing _pointer_types4.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc pointer_types4.c
reader@hacking:~/booksrc $ ./a.out
[char pointer] points to 0xbffff810, which contains the char 'a'
[char pointer] points to 0xbffff811, which contains the char 'b'
[char pointer] points to 0xbffff812, which contains the char 'c'
[char pointer] points to 0xbffff813, which contains the char 'd'
[char pointer] points to 0xbffff814, which contains the char 'e'
[integer pointer] points to 0xbffff7f0, which contains the integer 1
[integer pointer] points to 0xbffff7f4, which contains the integer 2
[integer pointer] points to 0xbffff7f8, which contains the integer 3
[integer pointer] points to 0xbffff7fc, which contains the integer 4
[integer pointer] points to 0xbffff800, which contains the integer 5
reader@hacking:~/booksrc $
</pre>

The compilation and output of this _pointer_types4.c_ is basically the same as that for _pointer_types3.c_. The void pointer is really just holding the memory addresses, while the hard-coded typecasting is telling the compiler to use the proper types whenever the pointer is used.

Since the type is taken care of by the typecasts, the void pointer is truly nothing more than a memory address. With the data types defined by typecasting, anything that is big enough to hold a four-byte value can work the same way as a void pointer. In _pointer_types5._, an unsigned integer is used to store this address.

__pointer_types5.__

```c
#include <stdio.h>

int main() 
{
    int i;

    char char_array[5] = {'a', 'b', 'c', 'd', 'e'};
    int int_array[5] = {1, 2, 3, 4, 5};

    unsigned int hacky_nonpointer;

    hacky_nonpointer = (unsigned int) char_array;

    // Iterate through the int array with the int_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[hacky_nonpointer] points to %p, which contains the char '%c'\n",
            hacky_nonpointer, *((char*) hacky_nonpointer));
        hacky_nonpointer = hacky_nonpointer + sizeof(char);
    }

    hacky_nonpointer = (unsigned int) int_array;

    // Iterate through the int array with the int_pointer.
    for (i = 0; i < 5; i++) 
    {
        printf("[hacky_nonpointer] points to %p, which contains the integer %d\n",
            hacky_nonpointer, *((int*) hacky_nonpointer));
        hacky_nonpointer = hacky_nonpointer + sizeof(int);
    }
}
```

This is rather hacky, but since this integer value is typecast into the proper pointer types when it is assigned and dereferenced, the end result is the same. Notice that instead of typecasting multiple times to do pointer arithmetic on an unsigned integer (which isn't even a pointer), the _sizeof()_ function is used to achieve the same result using normal arithmetic.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc pointer_types5.c
reader@hacking:~/booksrc $ ./a.out
[hacky_nonpointer] points to 0xbffff810, which contains the char 'a'
[hacky_nonpointer] points to 0xbffff811, which contains the char 'b'
[hacky_nonpointer] points to 0xbffff812, which contains the char 'c'
[hacky_nonpointer] points to 0xbffff813, which contains the char 'd'
[hacky_nonpointer] points to 0xbffff814, which contains the char 'e'
[hacky_nonpointer] points to 0xbffff7f0, which contains the integer 1
[hacky_nonpointer] points to 0xbffff7f4, which contains the integer 2
[hacky_nonpointer] points to 0xbffff7f8, which contains the integer 3
[hacky_nonpointer] points to 0xbffff7fc, which contains the integer 4
[hacky_nonpointer] points to 0xbffff800, which contains the integer 5
reader@hacking:~/booksrc $
</pre>

The important thing to remember about variables in C is that the compiler is the only thing that cares about a variable's type. In the end, after the program has been compiled, the variables are nothing more than memory addresses. This means that variables of one type can easily be coerced into behaving like another type by telling the compiler to typecast them into the desired type.