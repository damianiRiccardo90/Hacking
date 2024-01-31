# *__Getting your Hands Dirty__*

Now that the syntax of C feels more familiar and some fundamental programming concepts have been explained, actually programming in C isn't that big of a step. C compilers exist for just about every operating system and processor architecture out there, but for this book, Linux and an x86-based processor will be used exclusively. Linux is a free operating system that everyone has access to, and x86-based processors are the most popular consumer-grade processor on the planet. Since hacking is really about experimenting, it's probably best if you have a C compiler to follow along with.

Included with this book is a LiveCD you can use to follow along if your computer has an x86 processor. Just put the CD in the drive and reboot your computer. It will boot into a Linux environment without modifying your existing operating system. From this Linux environment you can follow along with the book and experiment on your own.

Let's get right to it. The firstprog.c program is a simple piece of C code that will print "Hello, world!" 10 times.

__firstprog.c__
```c
#include <stdio.h>

int main()
{
    int i;
    for(i = 0; i < 10; i++)           // Loop 10 times,
    {
        puts("Hello, world!\n"); // put the string to the output
    }
    return 0;                         // Tell OS the program exited without errors.
}
```

The main execution of a C program begins in the aptly named _main()_ function. Any text following two forward slashes (_//_) is a comment, which is ignored by the compiler.

The first line may be confusing, but it's just C syntax that tells the compiler to include header for a standard input/output (I/O) library named _stdio_. This header file is added to the program when it is compiled. It is located at _/usr/include/stdio.h_, and it defines several constants and function prototypes for corresponding functions in the standard I/O library. Since the _main()_ function uses the _printf()_ function from the standard I/O library, a function prototype is needed for _printf()_ before it can be used. This function prototype (along with many others) is included in the stdio.h header file. A lot of the power of C comes from its extensibility and libraries. The rest of the code should make sense and look a lot like the pseudo-code from before. You may have even noticed that there's a set of curly braces that can be eliminated. It should be fairly obvious what this program will do, but let's compile it using GCC and run it just to make sure.

The _GNU Compiler Collection_ (__GCC__) is a free C compiler that translates C into machine language that a processor can understand. The outputted translation is an executable binary file, which is called _a.out_ by default. Does the compiled program do what you thought it would?

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc firstprog.c
reader@hacking:~/booksrc $ ls -l a.out
-rwxr-xr-x 1 reader reader 6621 2007-09-06 22:16 a.out
reader@hacking:~/booksrc $ ./.a.out
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
Hello, world!
reader@hacking:~/booksrc $
</pre>