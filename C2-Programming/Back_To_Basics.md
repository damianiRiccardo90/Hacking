# *__Back to Basics__*

Now that the idea of programming is less abstract, there are a few other important concepts to know about C. Assembly language and computer processors existed before higher-level programming languages, and many modern programming concepts have evolved through time. In the same way that knowing a little about Latin can greatly improve one's understanding of the English language, knowledge of low-level programming concepts can assist the comprehension of higher-level ones. When continuing to the next section, remember that C code must be compiled into machine instructions before it can do anything.

### *__Strings__*

The value _"Hello, world!\n"_ passed to the _printf()_ function in the previous program is a string, technically, a character array. In C, an __array__ is simply a list of _n_ elements of a specific data type. A 20-character array is simply 20 adjacent characters located in memory. Arrays are also referred to as _buffers_. The _char_array.c_ program is an example of a character array.

__char_array.c__

```c
#include <stdio.h>

int main()
{
    char str_a[20];
    str_a[0] = 'H';
    str_a[1] = 'e';
    str_a[2] = 'l';
    str_a[3] = 'l';
    str_a[4] = 'o';
    str_a[5] = ',';
    str_a[6] = ' ';
    str_a[7] = 'W';
    str_a[8] = 'o';
    str_a[9] = 'r';
    str_a[10] = 'l';
    str_a[11] = 'd';
    str_a[12] = '!';
    str_a[13] = '\n';
    str_a[14] = '0';
    printf(str_a);
}
```

The GCC compiler can also be given the __-o__ switch to define the output file to compile to. This switch is used below to compile the program into an executable binary called __char_array__.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o char_array char_array.c
reader@hacking:~/booksrc $ ./char_array
Hello, world!
reader@hacking:~/booksrc $
</pre>

In the preceding program, a 20-element character array is defined as __str_a__, and each element of the array is written to, one by one. Notice that the number begins at 0, as opposed to 1. Also notice that the last character is a 0. (This is also called a _null byte_.) The character array was defined, so 20 bytes are allocated for it, but only 12 of these bytes are actually used. The null byte at the end is used as a delimiter character to tell any function that is dealing with the string to stop operations right there. The remaining extra bytes are just garbage and will be ignored. If a null byte is inserted in the fifth element of the character array, only the characters _Hello_ would be printed by the _printf()_ function.

Since setting each character in a character array is painstaking and strings are used fairly often, a set of standard functions was created for string manipulation. For example, the _strcpy()_ function will copy a string from a source to a destination, iterating through the source string and copying each byte to the destination (and stopping after it copies the null termination byte). The order of the function's arguments is similar to Intel assembly syntax: Destination first and then source. The _char_array.c_ program can be rewritten using _strcpy()_ to accomplish the same thing using the string library. The next version of the _char_array_ program shown below includes _string.h_ since it uses a string function.

__char_array2.c__

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str_a[20];

    strcpy(str_a, "Hello, world!\n");
    printf(str_a);
}
```

Let's take a look at this program with GDB. In the output below, the compiled program is opened with GDB and breakpoints are set before, in and after the _strcpy()_ call shown in bold. The debugger will pause the program at each breakpoint, giving us a chance to examine registers and memory. The _strcpy()_ function's code comes from a shared library, so the breakpoint in this function can't actually be set until the program is executed.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -g -o char_array2 char_array2.c
reader@hacking:~/booksrc $ gdb -q ./char_array2
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list
1   #include &lt;stdio.h&gt;
2   #include &lt;string.h&gt;
3
4   int main() {
5       char str_a[20];
<strong><em>6</em></strong>
<strong><em>7       strcpy(str_a, "Hello, world!\n");</em></strong>
<strong><em>8       printf(str_a);</em></strong>
9   }
(gdb) break 6
Breakpoint 1 at 0x80483c4: file char_array2.c, line 6.
(gdb) break strcpy
Function "strcpy" not defined.
Make breakpoint pending on future shared library load? (y or [n]) y
Breakpoint 2 (strcpy) pending.
(gdb) break 8
Breakpoint 3 at 0x80483d7: file char_array2.c, line 8.
(gdb)
</pre>

When the program is run, the _strcpy()_ breakpoint is resolved, At each breakpoint, we're going to look at _EIP_ and the instructions it points to. Notice that the memory location for _EIP_ at the middle breakpoint is different.

<pre style="color: white;">
(gdb) run
Starting program: /home/reader/booksrc/char_array2
Breakpoint 4 at 0xb7f076f4
Pending breakpoint "strcpy" resolved
Breakpoint 1, main () at char_array2.c:7
7 strcpy(str_a, "Hello, world!\n");
(gdb) i r eip
eip 0x80483c4 0x80483c4 &lt;main+16&gt;
(gdb) x/5i $eip
0x80483c4 &lt;main+16&gt;: mov DWORD PTR [esp+4],0x80484c4
0x80483cc &lt;main+24&gt;: lea eax,[ebp-40]
0x80483cf &lt;main+27&gt;: mov DWORD PTR [esp],eax
0x80483d2 &lt;main+30&gt;: call 0x80482c4 &lt;strcpy@plt&gt;
0x80483d7 &lt;main+35&gt;: lea eax,[ebp-40]
(gdb) continue
Continuing.

Breakpoint 4, 0xb7f076f4 in strcpy () from /lib/tls/i686/cmov/libc.so.6
(gdb) i r eip
<strong><em>eip 0xb7f076f4 0xb7f076f4 &lt;strcpy+4&gt;</em></strong>
(gdb) x/5i $eip
0xb7f076f4 &lt;strcpy+4&gt;: mov esi,DWORD PTR [ebp+8]
0xb7f076f7 &lt;strcpy+7&gt;: mov eax,DWORD PTR [ebp+12]
0xb7f076fa &lt;strcpy+10&gt;: mov ecx,esi
0xb7f076fc &lt;strcpy+12&gt;: sub ecx,eax
0xb7f076fe &lt;strcpy+14&gt;: mov edx,eax
(gdb) continue
Continuing.

Breakpoint 3, main () at char_array2.c:8
8 printf(str_a);
(gdb) i r eip
eip 0x80483d7 0x80483d7 &lt;main+35&gt;
(gdb) x/5i $eip
0x80483d7 &lt;main+35&gt;: lea eax,[ebp-40]
0x80483da &lt;main+38&gt;: mov DWORD PTR [esp],eax
0x80483dd &lt;main+41&gt;: call 0x80482d4 &lt;printf@plt&gt;
0x80483e2 &lt;main+46&gt;: leave
0x80483e3 &lt;main+47&gt;: ret
(gdb)
</pre>

The address in _EIP_ at the middle breakpoint is different because the code for the _strcpy()_ function comes from a loaded library. In fact, the debugger shows _EIP_ for the middle breakpoint in the _strcpy()_ function, while _EIP_ at the other two breakpoints is in the _main()_ function. I'd like to point out that _EIP_ is able to travel from the main code to the _strcpy()_ code and back again. Each time a function is called, a record is kept on a data structure simply called the stack. The _stack_ lets _EIP_ return through long chains of function calls. In GDB, the __bt__ command can be used to backtrace the stack. In the output below, the stack backtrace is shown at each breakpoint.

<pre style="color: white;">
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/reader/booksrc/char_array2
Error in re-setting breakpoint 4:
Function "strcpy" not defined.
Breakpoint 1, main () at char_array2.c:7
7 strcpy(str_a, "Hello, world!\n");
(gdb) bt
#0 main () at char_array2.c:7
(gdb) cont
Continuing.

Breakpoint 4, 0xb7f076f4 in strcpy () from /lib/tls/i686/cmov/libc.so.6
(gdb) bt
#0 0xb7f076f4 in strcpy () from /lib/tls/i686/cmov/libc.so.6
#1 0x080483d7 in main () at char_array2.c:7
(gdb) cont
Continuing.

Breakpoint 3, main () at char_array2.c:8
8 printf(str_a);
(gdb) bt
#0 main () at char_array2.c:8
(gdb)
</pre>

At the middle breakpoint, the backtrace of the stack shows its record the _strcpy()_ call. Also, you may notice that the _strcpy()_ function is at a slightly different address during the second run. This is due to an exploit protection method that is turned on by default in the Linux kernel since 2.6.11. We will tak about this protection in more detail later.

### *__Signed, Unsigned, Long, and Short__*

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

### *__Pointers__*

The _EIP_ register is a pointer that "points" to the current instruction during a program's execution by containing its memory address. The idea of pointers is used in C, also. Since the physical memory cannot actually be moved, the information in it must be copied. It can be very computationally expensive to copy large chunks of memory to be used by different functions or in different places. This is also expensive from a memory standpoint, since space for the new destination copy must be saved or allocated before the source can be copied. Pointers are a solution to this problem. Instead of copying a large block of memory, it is much simpler to pass around the address of the beginning of that block of memory.

Pointers in C can be defined and used like any other variable type. Since memory on the x86 architecture uses 32-bit addressing, pointers are also 32 bits in size (4 bytes). Pointers are defined by prepending an asterisk (*) to the variable name. Instead of defining a variable of that type, a pointer is  as something that points to data of that type. The _pointer.c_ program is an example of a pointer being used with the _char_ data type, which is only 1 byte in size.

__pointer.c__

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char str_a[20];  // A 20-element character array
    char* pointer;   // A pointer, meant for a character array
    char* pointer2;  // And yet another one

    strcpy(str_a, "Hello, world!\n");
    pointer = str_a; // Set the first pointer to the start of the array
    printf(pointer);

    pointer2 = pointer + 2; // Set the second one 2 bytes further in.
    printf(pointer2;)       // Print it.
    strcpy(pointer2, "y you guys!\n"); // Copy into that spot.
    printf(pointer);        // Print again.
}
```

As the comment in the code indicate, the first pointer is set at the beginning of the character array. When the character array is referenced like this, it is actually a pointer itself. This is how this buffer was passed as a pointer to the _printf()_ and _strcpy()_ functions earlier. The second pointer is set to the first pointer's address plus two, and then some things are printed (shown in the output below).

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o pointer pointer.c
reader@hacking:~/booksrc $ ./pointer
Hello, world!
llo, world!
Hey you guys!
reader@hacking:~/booksrc $
</pre>

Let's take a look at this with GDB. The program is recompiled, and a breakpoint is set on the tenth line of the source code. This will stop the program after the _"Hello, world!\n"_ string has been copied into the _str_a_ buffer and the pointer variable is set to the beginning of it.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -g -o pointer pointer.c
reader@hacking:~/booksrc $ gdb -q ./pointer
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list
1   #include &lt;stdio.h&gt;
2   #include &lt;string.h&gt;
3
4   int main() {
5       char str_a[20]; // A 20-element character array
6       char* pointer;  // A pointer, meant for a character array
7       char* pointer2; // And yet another one
8
9       strcpy(str_a, "Hello, world!\n");
10      pointer = str_a; // Set the first pointer to the start of the array.
(gdb)
11      printf(pointer);
12
13      pointer2 = pointer + 2; // Set the second one 2 bytes further in.
14      printf(pointer2); // Print it.
15      strcpy(pointer2, "y you guys!\n"); // Copy into that spot.
16      printf(pointer); // Print again.
17  }
(gdb) break 11
Breakpoint 1 at 0x80483dd: file pointer.c, line 11.
(gdb) run
Starting program: /home/reader/booksrc/pointer

Breakpoint 1, main () at pointer.c:11
11 printf(pointer);
(gdb) x/xw pointer
0xbffff7e0: 0x6c6c6548
(gdb) x/s pointer
0xbffff7e0: "Hello, world!\n"
(gdb)
</pre>

When the pointer is examined as a string, it's apparent that the given string is there and is located at memory address _0xbffff7e0_. Remember that the string itself isn't stored in the pointer variable, only the memory address _0xbffff7e0_ is stored there.

In order to see the actual data stored in the pointer variable, you must use the __address-of operator__. The _address-of operator_ is a _unary operator_, which simply means it operates on a single argument. This operator is just an ampersand (__&__) prepended to a variable name. When it's used, the address of that variable is returned, instead of the variable itself. This operator exists both in GDB and in the C programming language.

<pre style="color: white;">
(gdb) x/xw &pointer
0xbffff7dc: 0xbffff7e0
(gdb) print &pointer
$1 = (char **) 0xbffff7dc
(gdb) print pointer
$2 = 0xbffff7e0 "Hello, world!\n"
(gdb)
</pre>

When the _address-of operator_ is used, the pointer variable is shown to be located at the address _0xbffff7dc_ in memory, and it contains the address _0xbffff7e0_.

When the _address-of operator_ is often used in conjunction with pointers, since pointers contain memory addresses. The _addressof.c_ program demonstrates the _address-of operator_ being used to put the address of an integer variable into a pointer. This line is shown in bold below.

__addressof.c__

```c
#include <stdio.h>

int main()
{
    int int_var = 5;
    int* int_ptr;

    int_ptr = &int_var; // Put the address of int_var into int_ptr
}
```

The program itself doesn't actually output anything, but you can probably guess what happens, even before debugging with GDB.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -g addressof.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list
1   #include &lt;stdio.h&gt;
2
3   int main() {
4       int int_var = 5;
5       int* int_ptr;
6
7       int_ptr = &int_var; // Put the address of int_var into int_ptr.
8   }
(gdb) break 8
Breakpoint 1 at 0x8048361: file addressof.c, line 8.
(gdb) run
Starting program: /home/reader/booksrc/a.out

Breakpoint 1, main () at addressof.c:8
8   }
(gdb) print int_var
$1 = 5
(gdb) print &int_var
$2 = (int *) 0xbffff804
(gdb) print int_ptr
$3 = (int *) 0xbffff804
(gdb) print &int_ptr
$4 = (int **) 0xbffff800
(gdb)
</pre>

As usual, a breakpoint is set and the program is executed in the debugger. At this point the majority of the program has executed. The first _print_ command shows the value of _int_var_, and the second shows its address using the _address-of operator_. The next two print commands show that _int_ptr_ contains the address of _int_var_, and they also show the address of the _int_ptr_ for good measure.

An additional unary operator called the __dereference operator__ exists for use with pointers. This operator will return the data found in the address the pointer is pointing to, instead of the address itself. It takes the form of an asterisk in front of the variable name, similar to the declaration of a pointer. Once again, the _dereference operator_ exists both in GDB and in C. Used in GDB, it can retrieve the integer value _int_ptr_ points to.

<pre style="color: white;">
(gdb) print *int_ptr
$5 = 5
</pre>

A few additions to the _addressof.c_ code (shown in _addressof2.c_) will demonstrate all of these concepts. The added _printf()_ functions use format parameters, which I'll explain in the next section. For now, just focus on the program's output.

__addressof2.c__

```c
#include <stdio.h>

int main()
{
    int int_var = 5;
    int* int_ptr;

    int_ptr = &int_var; // Put the address of int_var into int_ptr

    printf("int_ptr = 0x%08x\n", int_ptr);
    printf("&int_ptr = 0x%08x\n", &int_ptr);
    printf("*int_ptr = 0x%08x\n\n", *int_ptr);

    printf("int_var is located at 0x%08x and contains %d\n", &int_var, int_var);
    printf("int_ptr is located at 0x%08x, contains 0x%08x, and points to %d\n\n",
        &int_ptr, int_ptr, *int_ptr);
}
```

The result of compiling and executing _addressof2.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc addressof2.c
reader@hacking:~/booksrc $ ./a.out
int_ptr = 0xbffff834
&int_ptr = 0xbffff830
*int_ptr = 0x00000005
int_var is located at 0xbffff834 and contains 5
int_ptr is located at 0xbffff830, contains 0xbffff834, and points to 5
reader@hacking:~/booksrc $
</pre>

When the unary operators are used with pointers, the _address-of operator_ can be thought of as moving backward, while the dereference operator moves forward in the direction the pointer is pointing.

### *__Format Strings__*

The _printf()_ function can be used to print more than just fixed strings. This function can also use format strings to print variables in many different formats. A _format string_ is just a character string with special escape sequences that tell the function to insert variables printed in a specific format in place of the escape sequence. The way the _printf()_ function has been used in the previous programs, the _"Hello, world!\n"_ string technically is the format string; however, it is devoid of special escape sequences. These _escape sequences_ are also called _format parameters_, and for each one found in the format string, the function is expected to take an additional argument. Each format parameter begins with a percent sign (_%_) and uses a single-character shorthand very similar to formatting characters used by GDB's examine command.

<div align="left" width="100%">
<img src="Value_Format_Parameters.png?raw=true" alt="Value Format Parameters" width="40%">
</div>

All of the preceding _format parameters_ receive their data as values, not pointers to values. There are also some format parameters that expect pointers, such as the following.

<div align="left" width="100%">
<img src="Pointer_Format_Parameters.png?raw=true" alt="Pointer Format Parameters" width="50%">
</div>

The __%s__ format parameter expects to be given a memory address; it prints the data at that memory address until a null byte is encountered. The __%n__ format parameter is unique in that it actually writes data. It also expects to be given a memory address, and it writes the number of bytes that have been written so far into that memory address.

For now, our focus will just be the format parameters used for displaying data. The _fmt_strings.c_ program shows some examples of different format parameters.

__fmt_strings.c__

```c
#include <stdio.h>

int main()
{
    char string[10];
    int A = -73;
    unsigned int B = 31337;

    strcpy(string, "sample");
    // Example of printing with different format string
    printf("[A] Dec: %d, Hex: %x, Unsigned: %u\n", A, A, A);
    printf("[B] Dec: %d, Hex: %x, Unsigned: %u\n", B, B, B);
    printf("[field width on B] 3: '%3u', 10: '%10u', '%08u'\n", B, B, B);
    printf("[string] %s Address %08x\n", string, string);
    // Example of unary address operator (dereferencing) and a %x format string
    printf("variable A is at address: %08x\n", &A);
}
```

In the preceding code, additional variable arguments are passed to each _printf()_ call for every format parameter in the format string. The final _printf()_ call uses the argument __&A__, which will provide the address of the variable __A__. The program's compilation and execution are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o fmt_strings fmt_strings.c
reader@hacking:~/booksrc $ ./fmt_strings
[A] Dec: -73, Hex: ffffffb7, Unsigned: 4294967223
[B] Dec: 31337, Hex: 7a69, Unsigned: 31337
[field width on B] 3: '31337', 10: '     31337', '00031337'
[string] sample Address bffff870
variable A is at address: bffff86c
reader@hacking:~/booksrc $
</pre>

The first two calls to _printf()_ demonstrate the printing of variables __A__ and __B__, using different format parameters. Since there are three format parameters in each line, the variables _A_ and _B_ need to be supplied three times each. The __%d__ format parameter allows for negative values, while __%u__ does not, since it is expecting unsigned values.

When the variable _A_ is printed using the _%u_ format parameter, it appears as a very high value. This is because _A_ is a negative number store in two's complement , and the format parameter is trying to print it as if it were an unsigned value. Since two's complement flips all the bits and adds one, the very high bits that used to be zero are now one.

The third line in the example, labeled _[field width on B]_, shows the use fo the field-width option in a format parameter. This is just an integer that designates the minimum field width for that format parameter. However, this is not a maximum field width, if the value to be outputted is greater than the field width, the field width will be exceeded. This happens when _3_ is used, since the output data needs 5 bytes. When _10_ is used as the field width, 5 bytes of blank space are outputted before the output data. Additionally, if a field width value begins with a 0, this means the field should be padded with zeroes. When _08_ is used, for example, the output is _00031337_.

The fourth line, labeled _[string]_, simply shows the use of the __%s__ format parameter. Remember that the variable string is actually a pointer containing the address of the string, which works out wonderfully, since the _%s_ format parameter expects its data to be passed by reference.

The final line just shows the address of the variable __A__, using the unary address operator to dereference the variable. This value is displayed as eight hexadecimal digits, padded by zeros.

As these examples show, you should use _%d_ for decimal, _%u_ for unsigned, and _%x_ for hexadecimal values. Minimum field widths can be set by putting a number right after the percent sign, and if the field width begins with 0, it will be padded with zeros. The _%s_ parameter can be used to print strings and should be passed the address of the string. So far, so good.

Format strings are used by an entire family of standard I/O functions, including _scanf()_, which basically works like _printf()_ but is used for input instead of output. One key difference is that the _scanf()_ function expects all of his arguments to be pointers, so the arguments must actually be variable addresses, not the variables themselves. This can be done using pointer variables or by using the unary address operator to retrieve the address of the normal variables. The _intput.c_ program and execution should help explain.

__input.c__

```c
#include <stdio.h>
#include <string.h>

int main()
{
    char message[10];
    int count, i;

    strcpy(message, "Hello, world!");

    printf("Repeat how many times? ");
    scanf("%d", &count);

    for (i = 0; i < count; i++)
        printf("%3d - %s\n", i, message);
}
```

In _input.c_, the _scanf()_ function is used to set the __count__ variable. The output below demonstrates its use.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o input input.c
reader@hacking:~/booksrc $ ./input
Repeat how many times? 3
    0 - Hello, world!
    1 - Hello, world!
    2 - Hello, world!
reader@hacking:~/booksrc $ ./input
Repeat how many times? 12
    0 - Hello, world!
    1 - Hello, world!
    2 - Hello, world!
    3 - Hello, world!
    4 - Hello, world!
    5 - Hello, world!
    6 - Hello, world!
    7 - Hello, world!
    8 - Hello, world!
    9 - Hello, world!
    10 - Hello, world!
    11 - Hello, world!
reader@hacking:~/booksrc $
</pre>

Format strings are used quite often, so familiarity with them is valuable. In addition, the ability to output the values of variables allows for debugging in the program, without the use of a debugger. Having some form of immediate feedback is fairly vital to the hacker's learning process, and something as simple as printing the value of a variable can allow for lots of exploitation,

### *__Typecasting__*

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

