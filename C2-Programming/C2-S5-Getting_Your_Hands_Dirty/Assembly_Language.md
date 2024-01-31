# *__Assembly Language__*

Since we are using Intel syntax assembly language for this book, our tools must be configured to use this syntax. Inside GDB, the disassembly syntax can be set to INtel by simply typing _set disassembly intel_ or _set dis intel_, for short. You can configure this setting to run every time GDB starts up by putting the command in the file __.gdbinit__ in your home directory.

<pre style="color: white;">
reader@hacking:~/booksrc $ gdb -q
(gdb) set dis intel
(gdb) quit
reader@hacking:~/booksrc $ echo "set dis intel" > ~/.gdbinit
reader@hacking:~/booksrc $ cat ~/.gdbinit
set dis intel
reader@hacking:~/booksrc $
</pre>

Now that GDB is configured to use Intel syntax, let's begin understanding it. The assembly instructions in Intel syntax generally follow this style:

<pre style="color: white;">
operation &lt;destination&gt;, &lt;source&gt;
</pre>

The destination and source values will either be a register, a memory address, or a value. The operations are usually intuitive mnemonics: The __mov__ operation will move a value from the source to the destination, __sub__ will subtract, __inc__ will increment, and so forth. For example, the instructions below will move the value from _ESP_ to _EBP_ and then subtract 8 from _ESP_ (storing the result in _ESP_).

<pre style="color: white;">
8048375: 89 e5    mov ebp,esp
8048377: 83 ec 08 sub esp,0x8
</pre>

There are also operations that are used to control the flow of execution. The __cmp__ operation is used to compare values, and basically any operation beginning with _j_ is used to jump to a different part of the code (depending on the result of the comparison). The example below first compares a 4-byte value located at _EBP_ minus 4 with the number 9. The next instruction is short-hand for _jump if less than or equal to_, referring to the result of the previous comparison. If that value is less than or equal to 9, execution jumps to the instruction at _0x8048393_. Otherwise, execution flows to the next instruction with an unconditional jump. If the value isn't less than or equal to 0, execution will jump to _0x80483a6_.

<pre style="color: white;">
804838b: 83 7d fc 09 cmp DWORD PTR [ebp-4],0x9
804838f: 7e 02       jle 8048393 &lt;main+0x1f&gt;
8048391: eb 13       jmp 80483a6 &lt;main+0x32&gt;
</pre>

These examples have been from our previous disassembly, and we have our debugger configured to use Intel syntax, so let's use the debugger to step through the first program at the assembly instruction level.

The __-g__ flag can be used by the GCC compiler to include extra debugging information, which will give GDB access to the source code.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -g firstprog.c
reader@hacking:~/booksrc $ ls -l a.out
-rwxr-xr-x 1 matrix users 11977 Jul 4 17:29 a.out
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/libthread_db.so.1".
(gdb) list
1   #include &lt;stdio.h&gt;
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
0x08048384 &lt;main+0&gt;:  push ebp
0x08048385 &lt;main+1&gt;:  mov ebp,esp
0x08048387 &lt;main+3&gt;:  sub esp,0x8
0x0804838a &lt;main+6&gt;:  and esp,0xfffffff0
0x0804838d &lt;main+9&gt;:  mov eax,0x0
0x08048392 &lt;main+14&gt;: sub esp,eax
<strong><em>0x08048394 &lt;main+16&gt;: mov DWORD PTR [ebp-4],0x0</em></strong>
0x0804839b &lt;main+23&gt;: cmp DWORD PTR [ebp-4],0x9
0x0804839f &lt;main+27&gt;: jle 0x80483a3 &lt;main+31&gt;
0x080483a1 &lt;main+29&gt;: jmp 0x80483b6 &lt;main+50&gt;
0x080483a3 &lt;main+31&gt;: mov DWORD PTR [esp],0x80484d4
0x080483aa &lt;main+38&gt;: call 0x80482a8 &lt;_init+56&gt;
0x080483af &lt;main+43&gt;: lea eax,[ebp-4]
0x080483b2 &lt;main+46&gt;: inc DWORD PTR [eax]
0x080483b4 &lt;main+48&gt;: jmp 0x804839b &lt;main+23&gt;
0x080483b6 &lt;main+50&gt;: leave
0x080483b7 &lt;main+51&gt;: ret
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
</pre>

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

<pre style="color: white;">
(gdb) i r eip
eip 0x8048384 0x8048384 &lt;main+16&gt;
(gdb) x/o 0x8048384
0x8048384 &lt;main+16&gt;: 077042707
(gdb) x/x $eip
0x8048384 &lt;main+16&gt;: 0x00fc45c7
(gdb) x/u $eip
0x8048384 &lt;main+16&gt;: 16532935
(gdb) x/t $eip
0x8048384 &lt;main+16&gt;: 00000000111111000100010111000111
(gdb)
</pre>

The memory the _EIP_ register is pointing to can be examined by using the address store in _EIP_. The debugger lets you reference registers directly, so _$eip_ is equivalent to the value _EIP_ contains at that moment. The value _077042707_ in octal is the same as _0x00fc45c7_ in hexadecimal, which is the same as _16532935_ in base-10 decimal, which in turn is the same as _00000000111111000100010111000111_ in binary. A number can also be prepended to the format of the examine command to examine multiple units at the target address.

<pre style="color: white;">
(gdb) x/2x $eip
0x8048384 &lt;main+16&gt;: 0x00fc45c7 0x83000000
(gdb) x/12x $eip
0x8048384 &lt;main+16&gt;: 0x00fc45c7 0x83000000 0x7e09fc7d 0xc713eb02
0x8048394 &lt;main+32&gt;: 0x84842404 0x01e80804 0x8dffffff 0x00fffc45
0x80483a4 &lt;main+48&gt;: 0xc3c9e5eb 0x90909090 0x90909090 0x5de58955
(gdb)
</pre>

The default size of a single unit is four-byte unit called a _word_. The size of the display units for the examine command can be changed by adding a size letter to the end of the format letter. The valid size letters are as follows:

<ul>
  <li><strong>b:</strong> A single byte</li>
  <li><strong>h:</strong> A halfword, which is two bytes in size</li>
  <li><strong>w:</strong> A word, which is four bytes in size</li>
  <li><strong>g:</strong> A giant, which is eight bytes in size</li>
</ul>

This is slightly confusing, because sometimes the term _word_ also refers to 2-byte values. In this case a _double word_ or _DWORD_ refers to a 4-byte value (dio boia). In this book, words and _DWORDs_ both refer to 4-byte values. If I'm talking about a 2-byte value, I'll call it a _short_ or a halfword. The following GDB output shows memory displayed in various sizes.

<pre style="color: white;">
(gdb) x/8xb $eip
0x8048384 &lt;main+16&gt;: 0xc7 0x45 0xfc 0x00 0x00 0x00 0x00 0x83
(gdb) x/8xh $eip
0x8048384 &lt;main+16&gt;: 0x45c7 0x00fc 0x0000 0x8300 0xfc7d 0x7e09 0xeb02 0xc713
(gdb) x/8xw $eip
0x8048384 &lt;main+16&gt;: 0x00fc45c7 0x83000000 0x7e09fc7d 0xc713eb02
0x8048394 &lt;main+32&gt;: 0x84842404 0x01e80804 0x8dffffff 0x00fffc45
(gdb)
</pre>

If you look closely, you may notice something odd about the data above. The first examine command shows the first eight bytes, and naturally, the examine commands that use bigger units display more data in total. However, the first examine shows the first two bytes to be _0xc7_ and _0x45_, but when a halfword is examined at the exact same memory address, the value _0x45c7_ is shown, with the bytes reversed. This same byte-reversal effect can be see when a full four-byte word is shown as _0x00fc45c7_, but when the first four bytes are shown byte by byte, they are in the order of _0xc7_, 0x45_, _0xfc_, and _0x00_.

This is because on the x86 processor values are stored in _little-endian byte order_, which means the least significant byte is stored first. For example, if four bytes are to be interpreted as a single value, the bytes must be used in reverse order. The GDB debugger is smart enough to know how values are stored, so when a word or halfword is examined, the byte must be reversed to display the correct values in hexadecimal. Revisiting these values displayed both as hexadecimal and unsigned decimals might help clear up any confusion.

<pre style="color: white;">
(gdb) x/4xb $eip
0x8048384 &lt;main+16&gt;: 0xc7 0x45 0xfc 0x00
(gdb) x/4ub $eip
0x8048384 &lt;main+16&gt;: 199 69 252 0
(gdb) x/1xw $eip
0x8048384 &lt;main+16&gt;: 0x00fc45c7
(gdb) x/1uw $eip
0x8048384 &lt;main+16&gt;: 16532935
(gdb) quit
The program is running. Exit anyway? (y or n) y
reader@hacking:~/booksrc $ bc -ql
199*(256^3) + 69*(256^2) + 252*(256^1) + 0*(256^0)
3343252480
0*(256^3) + 252*(256^2) + 69*(256^1) + 199*(256^0)
16532935
quit
<strong><em>reader@hacking:~/booksrc $</em></strong>
</pre>

The first four bytes are shown both in hexadecimal and standard unsigned decimal notation. A command-line calculator program called __bc__ is used to show that if the bytes are interpreted in the incorrect order, a horribly incorrect value of _3343252480_ is the result. The byte oder of a given architecture is an important detail to be aware of. While most debugging tools and compilers will take care of the details of byte order automatically, eventually you will directly manipulate memory by yourself.

In addition to converting byte order, GDB can do other conversions with the examine command. We've already seen that GDB can disassemble machine language instructions into human-readable assembly instructions. The examine command also accepts the format letter __i__, short for _instruction_, to display the memory as disassembled assembly language instructions.

<pre style="color: white;">
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) break main
Breakpoint 1 at 0x8048384: file firstprog.c, line 6.
(gdb) run
Starting program: /home/reader/booksrc/a.out
Breakpoint 1, main () at firstprog.c:6
6 for(i=0; i < 10; i++)
(gdb) i r $eip
eip 0x8048384 0x8048384 &lt;main+16&gt;
(gdb) x/i $eip
0x8048384 &lt;main+16&gt;: mov DWORD PTR [ebp-4],0x0
(gdb) x/3i $eip
0x8048384 &lt;main+16&gt;: mov DWORD PTR [ebp-4],0x0
0x804838b &lt;main+23&gt;: cmp DWORD PTR [ebp-4],0x9
0x804838f &lt;main+27&gt;: jle 0x8048393 &lt;main+31&gt;
(gdb) x/7xb $eip
0x8048384 &lt;main+16&gt;: 0xc7 0x45 0xfc 0x00 0x00 0x00 0x00
(gdb) x/i $eip
0x8048384 &lt;main+16&gt;: mov DWORD PTR [ebp-4],0x0
(gdb)
</pre>

In the output above, the _a.out_ program is run in GDB, with a breakpoint set at _main()_. Since the _EIP_ register is pointing to memory that actually contains machine language instructions, they disassemble quite nicely.

The previous _objdump_ disassembly confirms that the seven bytes _EIP_ is pointing to actually are machine language for the corresponding assembly instruction.

<pre style="color: white;">
8048384: c7 45 fc 00 00 00 00 mov DWORD PTR [ebp-4],0x0
</pre>

This assembly instruction will move the value of 0 into memory located at the address store in the _EBP_ register, minus 4. This is where the C variable __i__ is stored in memory; _i_ was declared as an integer that uses 4 bytes of memory on the x86 processor. Basically, this command will zero out the variable _i_ for the loop. If that memory is examined right now, it will contain nothing but random garbage. The memory at this location can be examined several different ways.

<pre style="color: white;">
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
</pre>

The _EBP_ register is shown to contain the address _0xbffff808_, and the assembly instruction will be writing to a value offset by 4 less than that, _0xbffff804_. The examine command can examine this memory address directly or by doing the math on the fly. The _print_ command can also be used to do simple math, but the result is stored in a temporary variable in the debugger. This variable named __$1__ can be used later to quickly re-access a particular location in memory. Any of the methods sown above will accomplish the same task: Displaying the 4 garbage bytes found in memory that will be zeroed out when the current instruction executes.

Let's execute the current instruction using the command _nexti_, which is short for _next instruction_. The processor will read the instruction at _EIP_, execute it, and advance _EIP_ to the next instruction.

<pre style="color: white;">
(gdb) nexti
0x0804838b 6 for(i=0; i < 10; i++)
(gdb) x/4xb $1
0xbffff804: 0x00 0x00 0x00 0x00
(gdb) x/dw $1
0xbffff804: 0
(gdb) i r eip
eip 0x804838b 0x804838b &lt;main+23&gt;
(gdb) x/i $eip
0x804838b &lt;main+23&gt;: cmp DWORD PTR [ebp-4],0x9
(gdb)
</pre>

As predicted, the previous command zeroes out the 4 bytes found at _EBP_ minus 4, which is memory set aside for the C variable _i_. Then _EIP_ advances to the next instruction. The next few instructions actually make more sense to talk about in a group.

<pre style="color: white;">
(gdb) x/10i $eip
0x804838b &lt;main+23&gt;: cmp DWORD PTR [ebp-4],0x9
0x804838f &lt;main+27&gt;: jle 0x8048393 &lt;main+31&gt;
0x8048391 &lt;main+29&gt;: jmp 0x80483a6 &lt;main+50&gt;
<strong><em>0x8048393 &lt;main+31&gt;: mov DWORD PTR [esp],0x8048484</em></strong>
0x804839a &lt;main+38&gt;: call 0x80482a0 &lt;printf@plt&gt;
0x804839f &lt;main+43&gt;: lea eax,[ebp-4]
0x80483a2 &lt;main+46&gt;: inc DWORD PTR [eax]
0x80483a4 &lt;main+48&gt;: jmp 0x804838b &lt;main+23&gt;
0x80483a6 &lt;main+50&gt;: leave
0x80483a7 &lt;main+51&gt;: ret
(gdb)
</pre>

The first instruction, _cmp_, is a compare instruction, which will compare the memory used by the C variable _i_ with the value 9. The next instruction, _jle_ stands for _jump if less than or equal to_. It uses the results of the previous comparison (which are actually stored in the _EFLAGS_ register) to jump _EIP_ to point to a different part of the code if the destination of the previous comparison operation is less than or equal to the source. In this case the instruction says to jump to the address _0x8048393_ if the value store in memory for the C variable _i_ is less than or equal to the value 9. If this isn't the case, the _EIP_ will continue to the next instruction, which is an unconditional jump instruction. This will cause the _EIP_ to jump to the address _0x80483a6_. These three instructions combine to create an if-then-else control structure: _If the i is less than or equal to 9, then go to the instruction at address 0x8048393; otherwise, go to the instruction at address 0x80483a6_. The first address of _0x8048393_ (shown in bold) is simply the instruction found after the fixed jump instruction, and the second address of _0x80483a6_ (shown in italics) is located at the end of the function.

Since we know the value 0 is stored in the memory location being compared with the value 9, and we know that 0 is less than or equal to 9, _EIP_ should be at _0x8048393_ after executing the next two instructions.

<pre style="color: white;">
(gdb) nexti
0x0804838f 6 for(i=0; i < 10; i++)
(gdb) x/i $eip
0x804838f &lt;main+27&gt;: jle 0x8048393 &lt;main+31&gt;
(gdb) nexti
8 printf("Hello, world!\n");
(gdb) i r eip
eip 0x8048393 0x8048393 &lt;main+31&gt;
(gdb) x/2i $eip
0x8048393 &lt;main+31&gt;: mov DWORD PTR [esp],0x8048484
0x804839a &lt;main+38&gt;: call 0x80482a0 &lt;printf@plt&gt;
(gdb)
</pre>

As expected, the previous two instructions let the program execution flow down to _0x8048393_, which brings us to the net two instructions. The first instruction is another _mov_ instruction that will write the address _0x8048484_ into the memory address contained in the _ESP_ register. But what is _ESP_ pointing to?

<pre style="color: white;">
(gdb) i r esp
esp 0xbffff800 0xbffff800
(gdb)
</pre>

Currently, _ESP_ points to the memory address _0xbffff800_, so when the _mov_ instruction is executed, the address _0x8048484_ is written there. But why? What's so special about the memory address _0x8048484_? There's one way to find out.

<pre style="color: white;">
(gdb) x/2xw 0x8048484
0x8048484: 0x6c6c6548 0x6f57206f
(gdb) x/6xb 0x8048484
0x8048484: 0x48 0x65 0x6c 0x6c 0x6f 0x20
(gdb) x/6ub 0x8048484
0x8048484: 72 101 108 108 111 32
(gdb)
</pre>

A trained eye might notice something about the memory here, in particular the range of the bytes. After examining memory for long enough, these types of visual patterns become more apparent. The bytes fall within the printable _ASCII_ range. ASCII is an agreed-upon standard that maps all the characters on your keyboard (and some that aren't) to fixed numbers. 

The bytes _0x48_, _0x65_, _0x6c_, and _0x6f_ all correspond to letters in the alphabet on the ASCII table shown below. This table is found in the man page for ASCC, available on most Unix systems by typing _man ascii_.

__ASCII Table__

|    Oct   |    Dec   |    Hex   |    Char    |    Oct   |    Dec   |    Hex   |    Char  |
|:--------:|:--------:|:--------:|:----------:|:--------:|:--------:|:--------:|:--------:|
|    000   |     0    |    00    | NUL '\0'   |    100   |    64    |    40    |     @    |
|    001   |     1    |    01    | SOH        |    101   |    65    |    41    |     A    |
|    002   |     2    |    02    | STX        |    102   |    66    |    42    |     B    |
|    003   |     3    |    03    | ETX        |    103   |    67    |    43    |     C    |
|    004   |     4    |    04    | EOT        |    104   |    68    |    44    |     D    |
|    005   |     5    |    05    | ENQ        |    105   |    69    |    45    |     E    |
|    006   |     6    |    06    | ACK        |    106   |    70    |    46    |     F    |
|    007   |     7    |    07    | BEL '\a'   |    107   |    71    |    47    |     G    |
|    010   |     8    |    08    | BS '\b'    | ***110***| ***72*** | ***48*** |  ***H*** |
|    011   |     9    |    09    | HT '\t'    |    111   |    73    |    49    |     I    |
|    012   |    10    |    0A    | LF '\n'    |    112   |    74    |    4A    |     J    |
|    013   |    11    |    0B    | VT '\v'    |    113   |    75    |    4B    |     K    |
|    014   |    12    |    0C    | FF '\f'    |    114   |    76    |    4C    |     L    |
|    015   |    13    |    0D    | CR '\r'    |    115   |    77    |    4D    |     M    |
|    016   |    14    |    0E    | SO         |    116   |    78    |    4E    |     N    |
|    017   |    15    |    0F    | SI         |    117   |    79    |    4F    |     O    |
|    020   |    16    |    10    | DLE        |    120   |    80    |    50    |     P    |
|    021   |    17    |    11    | DC1        |    121   |    81    |    51    |     Q    |
|    022   |    18    |    12    | DC2        |    122   |    82    |    52    |     R    |
|    023   |    19    |    13    | DC3        |    123   |    83    |    53    |     S    |
|    024   |    20    |    14    | DC4        |    124   |    84    |    54    |     T    |
|    025   |    21    |    15    | NAK        |    125   |    85    |    55    |     U    |
|    026   |    22    |    16    | SYN        |    126   |    86    |    56    |     V    |
|    027   |    23    |    17    | ETB        |    127   |    87    |    57    |     W    |
|    030   |    24    |    18    | CAN        |    130   |    88    |    58    |     X    |
|    031   |    25    |    19    | EM         |    131   |    89    |    59    |     Y    |
|    032   |    26    |    1A    | SUB        |    132   |    90    |    5A    |     Z    |
|    033   |    27    |    1B    | ESC        |    133   |    91    |    5B    |     [    |
|    034   |    28    |    1C    | FS         |    134   |    92    |    5C    |    \ '\\'|
|    035   |    29    |    1D    | GS         |    135   |    93    |    5D    |     ]    |
|    036   |    30    |    1E    | RS         |    136   |    94    |    5E    |     ^    |
|    037   |    31    |    1F    | US         |    137   |    95    |    5F    |     _    |
|    040   |    32    |    20    | SPACE      |    140   |    96    |    60    |     `    |
|    041   |    33    |    21    | !          |    141   |    97    |    61    |     a    |
|    042   |    34    |    22    | "          |    142   |    98    |    62    |     b    |
|    043   |    35    |    23    | #          |    143   |    99    |    63    |     c    |
|    044   |    36    |    24    | $          |    144   |   100    |    64    |     d    |
|    045   |    37    |    25    | %          | ***145***|***101*** | ***65*** |  ***e*** |
|    046   |    38    |    26    | &          |    146   |   102    |    66    |     f    |
|    047   |    39    |    27    | '          |    147   |   103    |    67    |     g    |
|    050   |    40    |    28    | (          |    150   |   104    |    68    |     h    |
|    051   |    41    |    29    | )          |    151   |   105    |    69    |     i    |
|    052   |    42    |    2A    | *          |    152   |   106    |    6A    |     j    |
|    053   |    43    |    2B    | +          |    153   |   107    |    6B    |     k    |
|    054   |    44    |    2C    | ,          | ***154***|***108*** | ***6C*** |  ***l*** |
|    055   |    45    |    2D    | -          |    155   |   109    |    6D    |     m    |
|    056   |    46    |    2E    | .          |    156   |   110    |    6E    |     n    |
|    057   |    47    |    2F    | /          | ***157***|***111*** | ***6F*** |  ***o*** |
|    060   |    48    |    30    | 0          |    160   |   112    |    70    |     p    |
|    061   |    49    |    31    | 1          |    161   |   113    |    71    |     q    |
|    062   |    50    |    32    | 2          |    162   |   114    |    72    |     r    |
|    063   |    51    |    33    | 3          |    163   |   115    |    73    |     s    |
|    064   |    52    |    34    | 4          |    164   |   116    |    74    |     t    |
|    065   |    53    |    35    | 5          |    165   |   117    |    75    |     u    |
|    066   |    54    |    36    | 6          |    166   |   118    |    76    |     v    |
|    067   |    55    |    37    | 7          |    167   |   119    |    77    |     w    |
|    070   |    56    |    38    | 8          |    170   |   120    |    78    |     x    |
|    071   |    57    |    39    | 9          |    171   |   121    |    79    |     y    |
|    072   |    58    |    3A    | :          |    172   |   122    |    7A    |     z    |
|    073   |    59    |    3B    | ;          |    173   |   123    |    7B    |     {    |
|    074   |    60    |    3C    | <          |    174   |   124    |    7C    |     \|   |
|    075   |    61    |    3D    | =          |    175   |   125    |    7D    |     }    |
|    076   |    62    |    3E    | >          |    176   |   126    |    7E    |     ~    |
|    077   |    63    |    3F    | ?          |    177   |   127    |    7F    |    DEL   |

Thankfully, GDB's examine command also contains provisions for looking at this type of memory. The __c__ format letter can be used to automatically look up a byte on the ASCII table, and the __s__ format letter will display an entire string of character data.

<pre style="color: white;">
(gdb) x/6cb 0x8048484
0x8048484: 72 'H' 101 'e' 108 'l' 108 'l' 111 'o' 32 ' '
(gdb) x/s 0x8048484
0x8048484: "Hello, world!\n"
(gdb)
</pre>

These commands reveal that the data string _"Hello, world!\n"_ is stored at memory address _0x8048484_. This string is the argument for the _printf()_ function, which indicates that moving the address of this string to the address store in _ESP_ (_0x8048484_) has something to do with this function. The following output shows the data string's address being moved into the address _ESP_ is pointing to.

<pre style="color: white;">
(gdb) x/2i $eip
0x8048393 &lt;main+31&gt;: mov DWORD PTR [esp],0x8048484
0x804839a &lt;main+38&gt;: call 0x80482a0 &lt;printf@plt&gt;
(gdb) x/xw $esp
0xbffff800: 0xb8000ce0
(gdb) nexti
0x0804839a 8 printf("Hello, world!\n");
(gdb) x/xw $esp
0xbffff800: 0x08048484
(gdb)
</pre>

The next instruction is actually called the _printf()_ function; it prints the data string. The previous instruction was setting up for the function call, and the results of the function call can be seen in the output below in bold.

<pre style="color: white;">
(gdb) x/i $eip
0x804839a &lt;main+38&gt;: call 0x80482a0 &lt;printf@plt&gt;
(gdb) nexti
<strong><em>Hello, world!</em></strong>
6 for(i=0; i < 10; i++)
(gdb)
</pre>

Continuing to use GDB to debug, let's examine the next two instructions. Once again, they make more sense to look at in a group.

<pre style="color: white;">
(gdb) x/2i $eip
0x804839f &lt;main+43&gt;: lea eax,[ebp-4]
0x80483a2 &lt;main+46&gt;: inc DWORD PTR [eax]
(gdb)
</pre>

These two instructions basically just increment the variable __i__ by 1. The __lea__ instruction is an acronym for _Load Effective Address_, which will load the familiar address of _EBP_ minus 4 into the _EAX_ register. The execution of this instruction is shown below.

<pre style="color: white;">
(gdb) x/i $eip
0x804839f &lt;main+43&gt;: lea eax,[ebp-4]
(gdb) print $ebp - 4
$2 = (void *) 0xbffff804
(gdb) x/x $2
0xbffff804: 0x00000000
(gdb) i r eax
eax 0xd 13
(gdb) nexti
0x080483a2 6 for(i=0; i < 10; i++)
(gdb) i r eax
eax 0xbffff804 -1073743868
(gdb) x/xw $eax
0xbffff804: 0x00000000
(gdb) x/dw $eax
0xbffff804: 0
(gdb)
</pre>

The following __inc__ instruction will increment the value found at this address (now stored in the _EAX_ register) by 1. The execution of this instruction is also shown below.

<pre style="color: white;">
(gdb) x/i $eip
0x80483a2 &lt;main+46&gt;: inc DWORD PTR [eax]
(gdb) x/dw $eax
0xbffff804: 0
(gdb) nexti
0x080483a4 6 for(i=0; i < 10; i++)
(gdb) x/dw $eax
0xbffff804: 1
(gdb)
</pre>

The end result is the value stored at the memory address _EBP_ minus 4 (_0xbffff804_), incremented by 1. This behavior corresponds to a portion of C code in which the variable _i_ is incremented in the for loop.
The next instruction is an unconditional jump instruction.

<pre style="color: white;">
(gdb) x/i $eip
0x80483a4 &lt;main+48&gt;: jmp 0x804838b &lt;main+23&gt;
(gdb)
</pre>

When this instruction is executed, it will send the program back to the instruction at address _0x804838b_. It does this by simply setting _EIP_ to that value. Looking at the full disassembly again, you should be able to tell which parts of the C code have been compiled into which machine instructions.

<pre style="color: white;">
(gdb) disass main
Dump of assembler code for function main:
0x08048374 &lt;main+0&gt;: push ebp
0x08048375 &lt;main+1&gt;: mov ebp,esp
0x08048377 &lt;main+3&gt;: sub esp,0x8
0x0804837a &lt;main+6&gt;: and esp,0xfffffff0
0x0804837d &lt;main+9&gt;: mov eax,0x0
0x08048382 &lt;main+14&gt;: sub esp,eax
0x08048384 &lt;main+16&gt;: mov DWORD PTR [ebp-4],0x0
0x0804838b &lt;main+23&gt;: cmp DWORD PTR [ebp-4],0x9
0x0804838f &lt;main+27&gt;: jle 0x8048393 &lt;main+31&gt;
0x08048391 &lt;main+29&gt;: jmp 0x80483a6 &lt;main+50&gt;
0x08048393 &lt;main+31&gt;: mov DWORD PTR [esp],0x8048484
0x0804839a &lt;main+38&gt;: call 0x80482a0 &lt;printf@plt&gt;
0x0804839f &lt;main+43&gt;: lea eax,[ebp-4]
0x080483a2 &lt;main+46&gt;: inc DWORD PTR [eax]
0x080483a4 &lt;main+48&gt;: jmp 0x804838b &lt;main+23&gt;
0x080483a6 &lt;main+50&gt;: leave
0x080483a7 &lt;main+51&gt;: ret
End of assembler dump.
(gdb) list
1   #include &lt;stdio.h&gt;
2
3   int main()
4   {
5     int i;
<strong><em>6     for(i=0; i < 10; i++)</em></strong>
7     {
<em>8       printf("Hello, world!\n");</em>
9     }
10  }
(gdb)
</pre>

The instructions shown in bold make up the for loop, and the instructions in italics are the _printf()_ call found within the loop. The program execution will jump back to the compare instruction, continue to execute the _printf()_ call, and increment the counter variable until it finally equals 10. At this point the conditional _jle_ instruction won't execute; instead, the instruction pointer will continue to the unconditional jump instruction, which exits the loop and ends the program.