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

The second four registers (_ESP_, _EBP_, _ESI_, and _EDI_) are also general purpose registers, but they are sometimes known as pointers and indexes. These stand for _Stack Pointer_, _Base Pointer_, _Source Index_, and _Destination Index_, respectively. The first two registers are called pointers because they store 32-bit addresses, which essentially point to that location in memory. These registers are fairly important to program execution and memory management; we will discuss them more later. The last two registers are also technically pointers, which are commonly used to point to the source and destination when data needs to be read from or written to. There are load and store instructions that use these registers, but for the most part, these registers can be thought of as just simple general-purpose registers.

The _EIP_ register is the _Instruction Pointer_ register, which points to the current instruction the processor is reading. Like a child pointing his finger at each word as he reads, the processor reads each instruction using the _EIP_ register as its finger. Naturally, this register is quite important and will be used a lot while debugging. Currently, it points to a memory address at _0x804838a_.

The remaining _EFLAGS_ register actually consists of several bit flags that are used for comparisons and memory segmentations. The actual memory is split into several different segments, which will be discussed later, and these registers keep track of that. For the most part, these registers can be ignored since they rarely need to be accessed directly.

### *__Assembly Language__*

Since we are using Intel syntax assembly language for this book, our tools must be configured to use this syntax. Inside GDB, the disassembly syntax can be set to INtel by simply typing _set disassembly intel_ or _set dis intel_, for short. You can configure this setting to run every time GDB starts up by putting the command in the file __.gdbinit__ in your home directory.

```
reader@hacking:~/booksrc $ gdb -q
(gdb) set dis intel
(gdb) quit
reader@hacking:~/booksrc $ echo "set dis intel" > ~/.gdbinit
reader@hacking:~/booksrc $ cat ~/.gdbinit
set dis intel
reader@hacking:~/booksrc $
```

Now that GDB is configured to use Intel syntax, let's begin understanding it. The assembly instructions in Intel syntax generally follow this style:

```
operation <destination>, <source>
```

The destination and source values will either be a register, a memory address, or a value. The operations are usually intuitive mnemonics: The __mov__ operation will move a value from the source to the destination, __sub__ will subtract, __inc__ will increment, and so forth. For example, the instructions below will move the value from _ESP_ to _EBP_ and then subtract 8 from _ESP_ (storing the result in _ESP_).

```
8048375: 89 e5    mov ebp,esp
8048377: 83 ec 08 sub esp,0x8
```

There are also operations that are used to control the flow of execution. The __cmp__ operation is used to compare values, and basically any operation beginning with _j_ is used to jump to a different part of the code (depending on the result of the comparison). The example below first compares a 4-byte value located at _EBP_ minus 4 with the number 9. The next instruction is short-hand for _jump if less than or equal to_, referring to the result of the previous comparison. If that value is less than or equal to 9, execution jumps to the instruction at _0x8048393_. Otherwise, execution flows to the next instruction with an unconditional jump. If the value isn't less than or equal to 0, execution will jump to _0x80483a6_.

```
804838b: 83 7d fc 09 cmp DWORD PTR [ebp-4],0x9
804838f: 7e 02       jle 8048393 <main+0x1f>
8048391: eb 13       jmp 80483a6 <main+0x32>
```

These examples have been from our previous disassembly, and we have our debugger configured to use Intel syntax, so let's use the debugger to step through the first program at the assembly instruction level.

The __-g__ flag can be used by the GCC compiler to include extra debugging information, which will give GDB access to the source code.

```
reader@hacking:~/booksrc $ gcc -g firstprog.c
reader@hacking:~/booksrc $ ls -l a.out
-rwxr-xr-x 1 matrix users 11977 Jul 4 17:29 a.out
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/libthread_db.so.1".
(gdb) list
1   #include <stdio.h>
2
3   int main()
4   {
5       int i;
6       for(i = 0; i < 10; i++)
7       {
8           printf("Hello, world!\n");
9       }
10  }
(gdb) disassemble main
Dump of assembler code for function main():
0x08048384 <main+0>:  push ebp
0x08048385 <main+1>:  mov ebp,esp
0x08048387 <main+3>:  sub esp,0x8
0x0804838a <main+6>:  and esp,0xfffffff0
0x0804838d <main+9>:  mov eax,0x0
0x08048392 <main+14>: sub esp,eax
0x08048394 <main+16>: mov DWORD PTR [ebp-4],0x0
0x0804839b <main+23>: cmp DWORD PTR [ebp-4],0x9
0x0804839f <main+27>: jle 0x80483a3 <main+31>
0x080483a1 <main+29>: jmp 0x80483b6 <main+50>
0x080483a3 <main+31>: mov DWORD PTR [esp],0x80484d4
0x080483aa <main+38>: call 0x80482a8 <_init+56>
0x080483af <main+43>: lea eax,[ebp-4]
0x080483b2 <main+46>: inc DWORD PTR [eax]
0x080483b4 <main+48>: jmp 0x804839b <main+23>
0x080483b6 <main+50>: leave
0x080483b7 <main+51>: ret
End of assembler dump.
(gdb) break main
Breakpoint 1 at 0x8048394: file firstprog.c, line 6.
(gdb) run
Starting program: /hacking/a.out
Breakpoint 1, main() at firstprog.c:6
6 for(i = 0; i < 10; i++)
(gdb) info register eip
eip 0x8048394 0x8048394
(gdb)
```

First, the source code is listed and the disassembly of the _main()_ function is displayed. Then a breakpoint is set at the start of _main()_, and the program is run. This breakpoint simply tells the debugger to pause the execution of the program when it gets to that point. Since the breakpoint hs been set at the start of the _main()_ function, the program hits the breakpoint and pauses before actually executing any instructions in _main()_. Then the value of _EIP_ (the Instruction Pointer) is displayed.

Notice that _EIP_ contains a memory address that points to an instruction in the _main()_ function's disassembly (shown in bold). The instructions before this (shown in italics) are collectively known as the _function prologue_ and are generated by the compiler to set up memory for the rest of the _main()_ function's local variables. Part of the reason variables need to be declared in C is to aid the construction of this section of code. The debugger knows this part of the code is automatically generated and is smart enough to skip over it. We'll talk more about the function prologue later, but for now we can take a cue from GDB and skip it.

The GDB debugger provides a direct method to examine memory, using the command __x__, which is short for examine. Examining memory is a critical skill for any hacker. Most hacker exploits are a lot like magic tricks, they seem amazing and magical, unless you know about sleight of and and misdirection. In both magic and hacking, if you were to look in just the right spot, the trick would be obvious. That's one of the reasons a good magician never does the same trick twice. But with a debugger like GDB, every aspect of a program's execution can be deterministically examined, paused, stepped through, and repeated as often as needed. Since a running program is mostly just a processor and segments of memory, examining memory is the first way to look at what's really going on.

The examine command in GDB can be used to look at a certain address of memory in a variety of ways. This command expects two arguments when it's used: The location in memory to examine and how to display that memory. The display format also uses a single-letter shorthand, which is optionally preceded by a count of how many items to examine. Some common format letters are as follows:

<ul>
  <li><strong>o:</strong> Display in octal.</li>
  <li><strong>x:</strong> Display in hexadecimal.</li>
  <li><strong>u:</strong> Display in unsigned, standard base-10 decimal.</li>
  <li><strong>t:</strong> Display in binary.</li>
</ul>

These can be used with the examine command to examine a certain memory address. In the following example, the current address of the _EIP_ register is used. Shorthand commands are often used with GDB, and even _info register eip_ can be shortened to just _i r eip_.

```
(gdb) i r eip
eip 0x8048384 0x8048384 <main+16>
(gdb) x/o 0x8048384
0x8048384 <main+16>: 077042707
(gdb) x/x $eip
0x8048384 <main+16>: 0x00fc45c7
(gdb) x/u $eip
0x8048384 <main+16>: 16532935
(gdb) x/t $eip
0x8048384 <main+16>: 00000000111111000100010111000111
(gdb)
```

The memory the _EIP_ register is pointing to can be examined by using the address store in _EIP_. The debugger lets you reference registers directly, so _$eip_ is equivalent to the value _EIP_ contains at that moment. The value _077042707_ in octal is the same as _0x00fc45c7_ in hexadecimal, which is the same as _16532935_ in base-10 decimal, which in turn is the same as _00000000111111000100010111000111_ in binary. A number can also be prepended to the format of the examine command to examine multiple units at the target address.

```
(gdb) x/2x $eip
0x8048384 <main+16>: 0x00fc45c7 0x83000000
(gdb) x/12x $eip
0x8048384 <main+16>: 0x00fc45c7 0x83000000 0x7e09fc7d 0xc713eb02
0x8048394 <main+32>: 0x84842404 0x01e80804 0x8dffffff 0x00fffc45
0x80483a4 <main+48>: 0xc3c9e5eb 0x90909090 0x90909090 0x5de58955
(gdb)
```

The default size of a single unit is four-byte unit called a _word_. The size of the display units for the examine command can be changed by adding a size letter to the end of the format letter. The valid size letters are as follows:

<ul>
  <li><strong>b:</strong> A single byte</li>
  <li><strong>h:</strong> A halfword, which is two bytes in size</li>
  <li><strong>w:</strong> A word, which is four bytes in size</li>
  <li><strong>g:</strong> A giant, which is eight bytes in size</li>
</ul>

This is slightly confusing, because sometimes the term _word_ also refers to 2-byte values. In this case a _double word_ or _DWORD_ refers to a 4-byte value (dio boia). In this book, words and _DWORDs_ both refer to 4-byte values. If I'm talking about a 2-byte value, I'll call it a _short_ or a halfword. The following GDB output shows memory displayed in various sizes.

```
(gdb) x/8xb $eip
0x8048384 <main+16>: 0xc7 0x45 0xfc 0x00 0x00 0x00 0x00 0x83
(gdb) x/8xh $eip
0x8048384 <main+16>: 0x45c7 0x00fc 0x0000 0x8300 0xfc7d 0x7e09 0xeb02 0xc713
(gdb) x/8xw $eip
0x8048384 <main+16>: 0x00fc45c7 0x83000000 0x7e09fc7d 0xc713eb02
0x8048394 <main+32>: 0x84842404 0x01e80804 0x8dffffff 0x00fffc45
(gdb)
```

If you look closely, you may notice something odd about the data above. The first examine command shows the first eight bytes, and naturally, the examine commands that use bigger units display more data in total. However, the first examine shows the first two bytes to be _0xc7_ and _0x45_, but when a halfword is examined at the exact same memory address, the value _0x45c7_ is shown, with the bytes reversed. This same byte-reversal effect can be see when a full four-byte word is shown as _0x00fc45c7_, but when the first four bytes are shown byte by byte, they are in the order of _0xc7_, 0x45_, _0xfc_, and _0x00_.

This is because on the x86 processor values are stored in _little-endian byte order_, which means the least significant byte is stored first. For example, if four bytes are to be interpreted as a single value, the bytes must be used in reverse order. The GDB debugger is smart enough to know how values are stored, so when a word or halfword is examined, the byte must be reversed to display the correct values in hexadecimal. Revisiting these values displayed both as hexadecimal and unsigned decimals might help clear up any confusion.

```
(gdb) x/4xb $eip
0x8048384 <main+16>: 0xc7 0x45 0xfc 0x00
(gdb) x/4ub $eip
0x8048384 <main+16>: 199 69 252 0
(gdb) x/1xw $eip
0x8048384 <main+16>: 0x00fc45c7
(gdb) x/1uw $eip
0x8048384 <main+16>: 16532935
(gdb) quit
The program is running. Exit anyway? (y or n) y
reader@hacking:~/booksrc $ bc -ql
199*(256^3) + 69*(256^2) + 252*(256^1) + 0*(256^0)
3343252480
0*(256^3) + 252*(256^2) + 69*(256^1) + 199*(256^0)
16532935
quit
reader@hacking:~/booksrc $
```

The first four bytes are shown both in hexadecimal and standard unsigned decimal notation. A command-line calculator program called __bc__ is used to show that if the bytes are interpreted in the incorrect order, a horribly incorrect value of _3343252480_ is the result. The byte oder of a given architecture is an important detail to be aware of. While most debugging tools and compilers will take care of the details of byte order automatically, eventually you will directly manipulate memory by yourself.

In addition to converting byte order, GDB can do other conversions with the examine command. We've already seen that GDB can disassemble machine language instructions into human-readable assembly instructions. The examine command also accepts the format letter __i__, short for _instruction_, to display the memory as disassembled assembly language instructions.

```
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) break main
Breakpoint 1 at 0x8048384: file firstprog.c, line 6.
(gdb) run
Starting program: /home/reader/booksrc/a.out
Breakpoint 1, main () at firstprog.c:6
6 for(i=0; i < 10; i++)
(gdb) i r $eip
eip 0x8048384 0x8048384 <main+16>
(gdb) x/i $eip
0x8048384 <main+16>: mov DWORD PTR [ebp-4],0x0
(gdb) x/3i $eip
0x8048384 <main+16>: mov DWORD PTR [ebp-4],0x0
0x804838b <main+23>: cmp DWORD PTR [ebp-4],0x9
0x804838f <main+27>: jle 0x8048393 <main+31>
(gdb) x/7xb $eip
0x8048384 <main+16>: 0xc7 0x45 0xfc 0x00 0x00 0x00 0x00
(gdb) x/i $eip
0x8048384 <main+16>: mov DWORD PTR [ebp-4],0x0
(gdb)
```

In the output above, the _a.out_ program is run in GDB, with a breakpoint set at _main()_. Since the _EIP_ register is pointing to memory that actually contains machine language instructions, they disassemble quite nicely.

The previous _objdump_ disassembly confirms that the seven bytes _EIP_ is pointing to actually are machine language for the corresponding assembly instruction.

```
8048384: c7 45 fc 00 00 00 00 mov DWORD PTR [ebp-4],0x0
```

This assembly instruction will move the value of 0 into memory located at the address store in the _EBP_ register, minus 4. This is where the C variable __i__ is stored in memory; _i_ was declared as an integer that uses 4 bytes of memory on the x86 processor. Basically, this command will zero out the variable _i_ for the loop. If that memory is examined right now, it will contain nothing but random garbage. The memory at this location can be examined several different ways.

```
(gdb) i r ebp
ebp 0xbffff808 0xbffff808
(gdb) x/4xb $ebp - 4
0xbffff804: 0xc0 0x83 0x04 0x08
(gdb) x/4xb 0xbffff804
0xbffff804: 0xc0 0x83 0x04 0x08
(gdb) print $ebp - 4
$1 = (void *) 0xbffff804
(gdb) x/4xb $1
0xbffff804: 0xc0 0x83 0x04 0x08
(gdb) x/xw $1
0xbffff804: 0x080483c0
(gdb)
```

The _EBP_ register is shown to contain the address _0xbffff808_, and the assembly instruction will be writing to a value offset by 4 less than that, _0xbffff804_. The examine command can examine this memory address directly or by doing the math on the fly. The _print_ command can also be used to do simple math, but the result is stored in a temporary variable in the debugger. This variable named __$1__ can be used later to quickly re-access a particular location in memory. Any of the methods sown above will accomplish the same task: Displaying the 4 garbage bytes found in memory that will be zeroed out when the current instruction executes.

Let's execute the current instruction using the command _nexti_, which is short for _next instruction_. The processor will read the instruction at _EIP_, execute it, and advance _EIP_ to the next instruction.

```
(gdb) nexti
0x0804838b 6 for(i=0; i < 10; i++)
(gdb) x/4xb $1
0xbffff804: 0x00 0x00 0x00 0x00
(gdb) x/dw $1
0xbffff804: 0
(gdb) i r eip
eip 0x804838b 0x804838b <main+23>
(gdb) x/i $eip
0x804838b <main+23>: cmp DWORD PTR [ebp-4],0x9
(gdb)
```

As predicted, the previous command zeroes out the 4 bytes found at _EBP_ minus 4, which is memory set aside for the C variable _i_. Then _EIP_ advances to the next instruction. The next few instructions actually make more sense to talk about in a group.

```
(gdb) x/10i $eip
0x804838b <main+23>: cmp DWORD PTR [ebp-4],0x9
0x804838f <main+27>: jle 0x8048393 <main+31>
0x8048391 <main+29>: jmp 0x80483a6 <main+50>
0x8048393 <main+31>: mov DWORD PTR [esp],0x8048484
0x804839a <main+38>: call 0x80482a0 <printf@plt>
0x804839f <main+43>: lea eax,[ebp-4]
0x80483a2 <main+46>: inc DWORD PTR [eax]
0x80483a4 <main+48>: jmp 0x804838b <main+23>
0x80483a6 <main+50>: leave
0x80483a7 <main+51>: ret
(gdb)
```

The first instruction, _cmp_, is a compare instruction, which will compare the memory used by the C variable _i_ with the value 9. The next instruction, _jle_ stands for _jump if less than or equal to_. It uses the results of the previous comparison (which are actually stored in the _EFLAGS_ register) to jump _EIP_ to point to a different part of the code if the destination of the previous comparison operation is less than or equal to the source. In this case the instruction says to jump to the address _0x8048393_ if the value store in memory for the C variable _i_ is less than or equal to the value 9. If this isn't the case, the _EIP_ will continue to the next instruction, which is an unconditional jump instruction. This will cause the _EIP_ to jump to the address _0x80483a6_. These three instructions combine to create an if-then-else control structure: _If the i is less than or equal to 9, then go to the instruction at address 0x8048393; otherwise, go to the instruction at address 0x80483a6_. The first address of _0x8048393_ (shown in bold) is simply the instruction found after the fixed jump instruction, and the second address of _0x80483a6_ (shown in italics) is located at the end of the function.

Since we know the value 0 is stored in the memory location being compared with the value 9, and we know that 0 is less than or equal to 9, _EIP_ should be at _0x8048393_ after executing the next two instructions.

```
(gdb) nexti
0x0804838f 6 for(i=0; i < 10; i++)
(gdb) x/i $eip
0x804838f <main+27>: jle 0x8048393 <main+31>
(gdb) nexti
8 printf("Hello, world!\n");
(gdb) i r eip
eip 0x8048393 0x8048393 <main+31>
(gdb) x/2i $eip
0x8048393 <main+31>: mov DWORD PTR [esp],0x8048484
0x804839a <main+38>: call 0x80482a0 <printf@plt>
(gdb)
```

As expected, the previous two instructions let the program execution flow down to _0x8048393_, which brings us to the net two instructions. The first instruction is another _mov_ instruction that will write the address _0x8048484_ into the memory address contained in the _ESP_ register. But what is _ESP_ pointing to?

```
(gdb) i r esp
esp 0xbffff800 0xbffff800
(gdb)
```