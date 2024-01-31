# *__Command-Line Arguments__*

Many nongraphical programs receive input in the form of command-line arguments. Unlike inputting with _scanf()_, command-line arguments don't require user interaction after the program has begun execution. This tends to be more efficient and is a useful input method.

In C, command-line arguments can be accessed in the _main()_ function by including two additional arguments to the function: An integer and a pointer to an array of strings. The integer will contain the number of arguments, and the array of strings will contain each of those arguments. The _commandline.c_ program and its execution should explain things.

__commandline.c__

```c
#include <stdio.h>

int main(int arg_count, char* arg_list[])
{
    int i;
    printf("There were %d arguments provided:\n", arg_count);
    for (i = 0; i < arg_count; i++)
    {
        printf("argument #%d\t-\t%s\n", i, arg_list[i]);
    }
}
```

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o commandline commandline.c
reader@hacking:~/booksrc $ ./commandline
There were 1 arguments provided:
argument #0 - ./commandline
reader@hacking:~/booksrc $ ./commandline this is a test
There were 5 arguments provided:
argument #0 - ./commandline
argument #1 - this
argument #2 - is
argument #3 - a
argument #4 - test
reader@hacking:~/booksrc $
</pre>

The zeroth argument is always the name of the executing binary, and the rest of the argument array (often called an _argument vector_) contains the remaining arguments as strings.

Sometimes a program will want to use a command-line argument as an integer as opposed to a string. Regardless of this, the argument is passed in as a string; however, there are standard conversion functions. Unlike simple typecasting, these functions can actually convert character arrays containing numbers into actual integers. The most common of these functions is __atoi()__, which is short for _ASCII to integer_. This function accepts a pointer to a string as its argument and returns the integer value it represents. Observe its usage in _convert.c_.

__convert.c__

```c
#include <stdio.h>

void usage(char* program_name) 
{
    printf("Usage: %s <message> <# of times to repeat>\n", program_name);
    exit(1);
}

int main(int argc, char* argv[])
{
    int i, count;

    if (argc < 3)       // If fewer than 3 arguments are used,
        usage(argv[0]); // display usage message and exit.
    
    count = atoi(argv[2]); // Convert the 2nd arg into an integer.
    printf("Repeating %d times..\n", count);

    for (i = 0; i < count; i++)
        printf("%3d - %s\n", i, argv[1]); // Print the 1st arg.
}
```

The result of compiling and executing _convert.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc convert.c
reader@hacking:~/booksrc $ ./a.out
Usage: ./a.out &lt;message&gt; &lt;# of times to repeat&gt;
reader@hacking:~/booksrc $ ./a.out 'Hello, world!' 3
Repeating 3 times..
0 - Hello, world!
1 - Hello, world!
2 - Hello, world!
reader@hacking:~/booksrc $
</pre>

In the preceding code, an _if_ statement makes sure that three arguments are used before these strings are accessed. If the program tries to access memory that doesn't exist or that the program doesn't have permission to read, the program will crash. In C it's important to check for these types of conditions and handle them in program logic. If the error-checking _if_ statement is commented out, this memory violation can be explored. The _convert2.c_ program should make this more clear.

__convert2.c__

```c
#include <stdio.h>

void usage(char* program_name) 
{
    printf("Usage: %s <message> <# of times to repeat>\n", program_name);
    exit(1);
}

int main(int argc, char* argv[])
{
    int i, count;

//  if(argc < 3)        // If fewer than 3 arguments are used,
//      usage(argv[0]); // display usage message and exit.

    count = atoi(argv[2]); // Convert the 2nd arg into an integer.
    printf("Repeating %d times..\n", count);

    for (i = 0; i < count; i++)
        printf("%3d - %s\n", i, argv[1]); // Print the 1st arg.
}
```

The results of compiling and executing _convert2.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc convert2.c
reader@hacking:~/booksrc $ ./a.out test
Segmentation fault (core dumped)
reader@hacking:~/booksrc $
</pre>

When the program isn't given enough command-line arguments, it still tries to access elements of the argument array, even though they don't exist. This results in the program crashing due to a segmentation fault.

Memory is split into segments (which will be discussed later), and some memory addresses aren't within the boundaries of the memory segments the program is given access to. WHen the program attempts to access an address that is out of bounds, it will crash and die in what's called a _segmentation fault_. This effect can be explored further with GDB.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -g convert2.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) run test
Starting program: /home/reader/booksrc/a.out test

Program received signal SIGSEGV, Segmentation fault.
0xb7ec819b in ?? () from /lib/tls/i686/cmov/libc.so.6
(gdb) where
#0 0xb7ec819b in ?? () from /lib/tls/i686/cmov/libc.so.6
#1 0xb800183c in ?? ()
#2 0x00000000 in ?? ()
(gdb) break main
Breakpoint 1 at 0x8048419: file convert2.c, line 14.
(gdb) run test
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /home/reader/booksrc/a.out test

Breakpoint 1, main (argc=2, argv=<strong><em>0xbffff894</em></strong>) at convert2.c:14
14 count = atoi(argv[2]); // convert the 2nd arg into an integer
(gdb) cont
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0xb7ec819b in ?? () from /lib/tls/i686/cmov/libc.so.6
(gdb) x/3xw 0xbffff894
0xbffff894: 0xbffff9b3 0xbffff9ce 0x00000000
(gdb) x/s 0xbffff9b3
0xbffff9b3: "/home/reader/booksrc/a.out"
(gdb) x/s 0xbffff9ce
0xbffff9ce: "test"
(gdb) x/s 0x00000000
0x0: &lt;Address 0x0 out of bounds&gt;
(gdb) quit
The program is running. Exit anyway? (y or n) y
reader@hacking:~/booksrc $
</pre>

The program is executed with a single command-line argument of _test_ within GDB, which causes the program to crash. The __where__ command will sometimes show a useful backtrace of the stack; however, in this case, the stack was too badly mangled in the crash. A breakpoint is set on main and the program is re-executed to get the value of the argument vector (shown in bold). Since the argument vector is a pointer to a list of strings, it is actually a pointer to a list of pointers. Using the command _x/3xw_ to examine the first three memory addresses stored at the argument vector's address shows that they are themselves pointers to strings. The first one is the zeroth argument, the second is the _test_ argument, and the third is zero, which is out of bounds. When the program tries to access this memory address, it crashes with a segmentation fault.