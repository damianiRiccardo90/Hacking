# *__The x86 Processor__*

The 8086 CPU was the first x86 processor. It was developed and manufactured by Intel, which later developed more advanced processors in the same family: The 80186, 80286, 80386, and 80486. If you remember people talking about 386 and 486 processors in the '80s and '90s, this is what they were referring to.

The x86 processor has several registers, which are like internal variables for the processor. I could just talk abstractly about these registers now, but I think it's always better to see things for yourself. The GNU development tools also include a debugger called __GDB__. _Debuggers_ are used by programmers to step through compiled programs, examine program memory, and view processor registers. A programmer who has never used a debugger to look at the inner workings of a program is like a seventeenth-century doctor who has never used a microscope. Similar to a microscope, a debugger allows a hacker to observe the microscopic world of machine code, but a debugger is far more powerful than this metaphor allows. Unlike a microscope, a debugger can view the execution from all angles, pause it, and change anything along the way.

Below, GDB is used to show the state of the processor registers right before the program starts.

<pre style="color: white;">
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
eip 0x804837a 0x804837a &lt;main+6&gt;
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
</pre>

A breakpoint is set on the _main()_ function so execution will stop right before our code is executed. Then GDB runs the program, stops at the breakpoint, and is told to display all the processor registers and their current states.

The first four registers (_EAX_, _ECX_, _EDX_, and _EBX_) are known as general purpose registers. These are called the _Accumulator_, _Counter_, _Data_, and _Base_ registers, respectively. They are used for a variety of purposes, but they mainly act as temporary variables for the CPU when it is executing machine instructions.

The second four registers (_ESP_, _EBP_, _ESI_, and _EDI_) are also general purpose registers, but they are sometimes known as pointers and indexes. These stand for _Stack Pointer_, _Base Pointer_, _Source Index_, and _Destination Index_, respectively. The first two registers are called pointers because they store 32-bit addresses, which essentially point to that location in memory. These registers are fairly important to program execution and memory management; we will discuss them more later. The last two registers are also technically pointers, which are commonly used to point to the source and destination when data needs to be read from or written to. There are load and store instructions that use these registers, but for the most part, these registers can be thought of as just simple general-purpose registers.

The _EIP_ register is the _Instruction Pointer_ register, which points to the current instruction the processor is reading. Like a child pointing his finger at each word as he reads, the processor reads each instruction using the _EIP_ register as its finger. Naturally, this register is quite important and will be used a lot while debugging. Currently, it points to a memory address at _0x804838a_.

The remaining _EFLAGS_ register actually consists of several bit flags that are used for comparisons and memory segmentations. The actual memory is split into several different segments, which will be discussed later, and these registers keep track of that. For the most part, these registers can be ignored since they rarely need to be accessed directly.