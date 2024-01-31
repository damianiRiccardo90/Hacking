# *__Strings__*

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