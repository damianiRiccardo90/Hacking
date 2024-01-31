# *__Format Strings__*

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

Format strings are used quite often, so familiarity with them is valuable. In addition, the ability to output the values of variables allows for debugging in the program, without the use of a debugger. Having some form of immediate feedback is fairly vital to the hacker's learning process, and something as simple as printing the value of a variable can allow for lots of exploitation.