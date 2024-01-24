# *__Getting your Hands Dirty__*

Now that the syntax of C feels more familiar and some fundamental programming concepts have been explained, actually programming in C isn't that big of a step. C compilers exist for just about every operating system and processor architecture out there, but for this book, Linux and an x86-based processor will be used exclusively. Linux is a free operating system that everyone has access to, and x86-based processors are the most popular consumer-grade processor on the planet. Since hacking is really about experimenting, it's probably best if you have a C compiler to follow along with.

Included with this book is a LiveCD you can use to follow along if your computer has an x86 processor. Just put the CD in the drive and reboot your computer. It will boot into a Linux environment without modifying your existing operating system. From this Linux environment you can follow along with the book and experiment on your own.

Let's get right to it. The firstprog.c program is a simple piece of C code that will print "Hell to the world!" 10 times.

__firstprog.c__
```c
#include <stdio.h>

int main()
{
    int i;
    for(i = 0; i < 10; i++)           // Loop 10 times,
    {
        puts("Hell to the world!\n"); // put the string to the output
    }
    return 0;                         // Tell OS the program exited without errors.
}
```

The main execution of a C program begins in the aptly named _main()_ function. Any text following two forward slashes (_//_) is a comment, which is ignored by the compiler.

The first line may be confusing, but it's just C syntax that tells the compiler to include header for a standard input/output (I/O) library named _stdio_. This header file is added to the program when it is compiled. It is located at _/usr/include/stdio.h_, and it defines several constants and function prototypes for corresponding functions in the standard I/O library. Since the _main()_ function uses the _printf()_ function from the standard I/O library, a function prototype is needed for _printf()_ before it can be used. This function prototype (along with many others) is included in the stdio.h header file. A lot of the power of C comes from its extensibility and libraries. The rest of the code should make sense and look a lot like the pseudo-code from before. You may have even noticed that there's a set of curly braces that can be eliminated. It should be fairly obvious what this program will do, but let's compile it using GCC and run it just to make sure.

The _GNU Compiler Collection_ (__GCC__) is a free C compiler that translates C into machine language that a processor can understand. The outputted translation is an executable binary file, which is called _a.out_ by default. Does the compiled program do what you thought it would?

```
reader@hacking:~/booksrc $ gcc firstprog.c
reader@hacking:~/booksrc $ ls -l a.out
-rwxr-xr-x 1 reader reader 6621 2007-09-06 22:16 a.out
reader@hacking:~/booksrc $ ./.a.out
Hell to the world!
Hell to the world!
Hell to the world!
Hell to the world!
Hell to the world!
Hell to the world!
Hell to the world!
Hell to the world!
Hell to the world!
Hell to the world!
reader@hacking:~/booksrc $
```

### *__The Bigger Picture__*

Okay, this has all been stuff you would learn in an elementary programming class, basic, but essential. Most introductory programming classes just teach how to read and write C. Don't get me wrong, being fluent in C is very useful and is enough to make you a decent programmer, but it's only a piece of the bigger picture. Most programmers learn the language from the top down and never see the big picture. Hackers get their edge from knowing how all the pieces interact within this bigger picture. To see the bigger picture in the real of programming, simply realize that C code is meant to be compiled. The code can't actually do anything until it's compiled into an executable binary file. Thinking of C-source as a program is a common misconception that is exploited byt hackers every day. The binary _a.out_'s instructions are written in machine language, an elementary language the CPU can understand. Compilers are designed to translate the language of C code into machine language for a variety of processor architectures. In this case, the processor is in a family that uses the x86 architecture. There are also Sparc processor architectures (used in Sun Workstations) and the PowerPC processor architecture (used in pre-Intel Macs). Each architecture has a different machine language, so the compiler acts as a middle ground, translating C code into machine language for the target architecture.

As long as the compiled program works, the average programmer is only concerned with source code. But a hacker realizes that the compiled program is what actually gets executed out in the real world. With a better understanding of how the CPU operates, a hacker can manipulate the programs that run on it. We have seen the source code for our first program and compiled it into an executable binary for x86 architecture. But what does this executable binary look like? The GNU development tools include a program called __objdump__, which can be used to examine compiled binaries. Let's start by looking at the machine code the _main()_ function was translated into.

```
reader@hacking:~/booksrc $ objdump -D a.out | grep -A20 main.:
08048374 <main>:
8048374: 55 push %ebp
8048375: 89 e5 mov %esp,%ebp
8048377: 83 ec 08 sub $0x8,%esp
804837a: 83 e4 f0 and $0xfffffff0,%esp
804837d: b8 00 00 00 00 mov $0x0,%eax
8048382: 29 c4 sub %eax,%esp
8048384: c7 45 fc 00 00 00 00 movl $0x0,0xfffffffc(%ebp)
804838b: 83 7d fc 09 cmpl $0x9,0xfffffffc(%ebp)
804838f: 7e 02 jle 8048393 <main+0x1f>
8048391: eb 13 jmp 80483a6 <main+0x32>
8048393: c7 04 24 84 84 04 08 movl $0x8048484,(%esp)
804839a: e8 01 ff ff ff call 80482a0 <printf@plt>
804839f: 8d 45 fc lea 0xfffffffc(%ebp),%eax
80483a2: ff 00 incl (%eax)
80483a4: eb e5 jmp 804838b <main+0x17>
80483a6: c9 leave
80483a7: c3 ret
80483a8: 90 nop
80483a9: 90 nop
80483aa: 90 nop
reader@hacking:~/booksrc $
```

The _objdump_ program will spit out far too many lines of output sensibly examine, so the output is piped into __grep__ with the command-line option to only display 20 lines after the regular expression _main.:_. Each byte is represented in _hexadecimal notation_, which is a base-16 numbering system. The numbering system you are most familiar with uses a base-10 system, since at 10 you need to add an extra symbol. Hexadecimal uses 0 through 9 to represent 0 through 9, but it also uses A through F to represent the values 10 through 15. This is a convenient notation since a byte has 256 ($ 2^8 $) possible values, so each byte can be described with 2 hexadecimal digits.

The hexadecimal numbers, starting with _0x8048374_ on the far left, are memory addresses. The bits of the machine language instructions must be put somewhere, and this somewhere is called _memory_. Memory is just a collection of bytes of temporary storage space that are numbered with addresses.

Like a row of houses on a local street, each with its own address, memory can be thought of as a row of bytes, each with its own memory address. Each byte of memory can be accessed by its address, and in this case the CPU accesses this part of memory to retrieve the machine language instructions that make up the compiled program. Older Intel x86 processors use a 32-bit addressing scheme, while newer ones use a 64-bit one. The 32-bit processors have $ 2^{32} $ or (4,294,967,296) possible addresses, while the 64-bit ones have $ 2^{64} $ (1.84467441 x $ 10^{39} $) possible addresses. The 64-bit processors can run in 32-bit compatibility mode, which allows them ro tun 32-bit code quickly.

The hexadecimal bytes in the middle of the listing above are the machine language instructions for the x86 processor. Of course, these hexadecimal values are only representations of the bytes of binary 1s and 0s the CPU can understand. But since _0101010110001001111001011000001111101100111100001 . . ._ isn't very useful to anything other than the processor, the machine code is displayed as hexadecimal bytes and each instruction is put on its own line, like splitting a paragraph into sentences.

Come to think of it, the hexadecimal bytes really aren't very useful themselves, either, that's where assembly language comes in. The instructions on the far right are in assembly language. Assembly language is really just a collection of mnemonics for the corresponding machine language instructions. The instruction __ret__ is far easier to remember and make sens of than _0xc3_ or _11000011_. Unlike C and other compiled languages, assembly language instructions have a direct one-to-one relationship with their corresponding machine language instructions. This means that since every processor architecture has different machine language instructions, each also has a different form of assembly language. Assembly is just a way for programmers to represent the machine language instructions that are given to the processor. Exactly how these machine language instructions are represented is simply a matter of convention and preference. While you can theoretically create your own x86 assembly language syntax, most people stick with one of the two main types: AT&T syntax and Intel syntax. The assembly shown in the output on page 21 is AT&T syntax, as just about all of Linux's disassembly tools use this syntax by default. It's easy to recognize AT&T syntax by the cacophony of % and $ symbols prefixing everything (take a look again at the example on page 21). The same code can be shown in Intel syntax by providing an additional command-line option, _-M intel_, to _objdump_, as shown in the output below.

```
reader@hacking:~/booksrc $ objdump -M intel -D a.out | grep -A20 main.:
08048374 <main>:
8048374: 55 push ebp
8048375: 89 e5 mov ebp,esp
8048377: 83 ec 08 sub esp,0x8
804837a: 83 e4 f0 and esp,0xfffffff0
804837d: b8 00 00 00 00 mov eax,0x0
8048382: 29 c4 sub esp,eax
8048384: c7 45 fc 00 00 00 00 mov DWORD PTR [ebp-4],0x0
804838b: 83 7d fc 09 cmp DWORD PTR [ebp-4],0x9
804838f: 7e 02 jle 8048393 <main+0x1f>
8048391: eb 13 jmp 80483a6 <main+0x32>
8048393: c7 04 24 84 84 04 08 mov DWORD PTR [esp],0x8048484
804839a: e8 01 ff ff ff call 80482a0 <printf@plt>
804839f: 8d 45 fc lea eax,[ebp-4]
80483a2: ff 00 inc DWORD PTR [eax]
80483a4: eb e5 jmp 804838b <main+0x17>
80483a6: c9 leave
80483a7: c3 ret
80483a8: 90 nop
80483a9: 90 nop
80483aa: 90 nop
reader@hacking:~/booksrc $
```

Personally, I think Intel syntax is much more readable and easier to understand, so for the purposes of this book, I will try to stick with this syntax. Regardless of the assembly language representation, the commands a processor understands are quite simple. These instructions consist of an operation and sometimes additional arguments that describe the destination and/or the source for the operation. These operations move memory around, perform some sort of basic math, or interrupt the processor to get it to do something else. In the end, that's all a computer processor can really do. But in the same way millions of books have been written using a relatively small alphabet of letters, an infinite number of possible programs can be create using a relatively small collection of machine instructions. The bigger picture keeps getting bigger...

### *__The x86 Processor__*

The 8086 CPU was the first x86 processor. It was developed and manufactured by Intel, which later developed more advanced processors in the same family: The 80186, 80286, 80386, and 80486. If you remember people talking about 386 and 486 processors in the '80s and '90s, this is what they were referring to.

The x86 processor has several registers, which are like internal variables for the processor. I could just talk abstractly about these registers now, but I think it's always better to see things for yourself. The GNU development tools also include a debugger called __GDB__. _Debuggers_ are used by programmers to step through compiled programs, examine program memory, and view processor registers. A programmer who has never used a debugger to look at the inner workings of a program is like a seventeenth-century doctor who has never used a microscope. Similar to a microscope, a debugger allows a hacker to observe the microscopic world of machine code, but a debugger is far more powerful than this metaphor allows. Unlike a microscope, a debugger can view the execution from all angles, pause it, and change anything along the way.

Below, GDB is used to show the state of the processor registers right before the program starts.

```
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) break main
Breakpoint 1 at 0x804837a
(gdb) run
Starting program: /home/reader/booksrc/a.out
Breakpoint 1, 0x0804837a in main ()
(gdb) info registers
eax 0xbffff894 -1073743724
ecx 0x48e0fe81 1222704769
edx 0x1 1
ebx 0xb7fd6ff4 -1208127500
esp 0xbffff800 0xbffff800
ebp 0xbffff808 0xbffff808
esi 0xb8000ce0 -1207956256
edi 0x0 0
eip 0x804837a 0x804837a <main+6>
eflags 0x286 [ PF SF IF ]
cs 0x73 115
ss 0x7b 123
ds 0x7b 123
es 0x7b 123
fs 0x0 0
gs 0x33 51
(gdb) quit
The program is running. Exit anyway? (y or n) y
reader@hacking:~/booksrc $
```

A breakpoint is set on the _main()_ function so execution will stop right before our code is executed. Then GDB runs the program, stops at the breakpoint, and is told to display all the processor registers and their current states.

The first four registers (_EAX_, _ECX_, _EDX_, and _EBX_) are known as general purpose registers. These are called the _Accumulator_, _Counter_, _Data_, and _Base_ registers, respectively. They are used for a variety of purposes, but they mainly act as temporary variables for the CPU when it is executing machine instructions.

The second four registers (_ESP_, _EBP_, _ESI_, and _EDI_) are also general purpose registers, but they are sometimes known as pointers and indexes. These stand for _Stack Pointer_, _Base Pointer_, _Source Index_, and _Destination Index_, respectively.