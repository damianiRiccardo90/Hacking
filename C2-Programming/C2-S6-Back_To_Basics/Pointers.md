# *__Pointers__*

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