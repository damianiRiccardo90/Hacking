# *__Memory Segmentation__*

A compiled program's memory is divided into five segments: Text, data, bss, heap, and stack. Each segment represents a special portion of memory that is set aside for a certain purpose.

The _text segment_ is also sometimes called the _code segment_. This is where the assembled machine language instructions of the program are located. The execution of instructions in this segment is nonlinear, thanks to the aforementioned high-level control structures and functions, which compile into branch, jump, and call instructions in assembly language. As a program executes, the _EIP_ is set to the first instruction in the the text segment. The processor then follows an execution loop that does the following:

1. Reads the instruction that _EIP_ is pointing to
2. Adds the byte length of the instruction to _EIP_
3. Executes the instruction that was read in step 1
4. Goes back to step 1

Sometimes the instruction will be a jump or a call instruction, which changes the _EIP_ to a different address of memory. The processor doesn't care about the change, because it's expecting the execution to be nonlinear anyway. If _EIP_ is changed in step 3, the processor will just go back to step 1 and read the instruction found at the address of whatever _EIP_ was changed to.

Write permission is disabled in the text segment, as it is not used to store variables, only code. This prevents people from actually modifying the program code; any attempt to write to this segment of memory will cause the program to alert the user that something bad happened, and the program will be killed. Another advantage of this segment being read-only is that is can be shared among different copies of the program, allowing multiple executions of the program at the same time without any problems. It should also be noted that this memory segment has a fixed size, since nothing ever changes in it.

The _data_ and _bss_ segments are used to store global and static program variables. The _data segment_ is filled with the initialized global and static variables while the _bss segment_ is filled with their uninitialized counterparts. Although these segments are writable, they also have a fixed size. Remember that global variables persist, despite the functional context (like the variable _j_ in the previous examples). Both global and static variables are able to persist because they are stored in their own memory segments.

The _heap segment_ is a segment of memory a programmer can directly control. Blocks of memory in this segment can be allocated and used for whatever the programmer might need.On notable point about the heap segment is that it isn't of fixed size, so it can grow larger or smaller as needed. All of the memory within the heap is managed by allocator and deallocator algorithms, which respectively reserve a region of memory in the heap for use and remove reservations to allow that portion of memory to be reused for later reservations. The heap will grow and shrink depending on how much memory is reserved for use. This means a programmer using the heap allocation functions can reserve and free memory on the fly. The growth of the heap moves downward toward higher memory addresses.

The _stack segment_ also has variable size and is used as a temporary scratch pad to store local function variables and context during function calls. This is what GDB's backtrace command looks at. When a program calls a function, that function will have its own set of passed variables, and the function's code will be at a different memory location in the text (or code) segment. Since the context and the _EIP_ must change when a function is called, the stack is used to remember all of the passed variables, the location the _EIP_ should return to after the function is finished, and all the local variables used by that function. All of this information is stored together on the stack in what is collectively called a _stack frame_. The stack contains many stack frames.

In general computer science terms, a _stack_ is an abstract data structure that is used frequently. It has _first-in_, _last-out_ (__FILO__) ordering, which means the first item that is put into a stack is the last item to come out of it. Think of it as putting beads on a pice of string that has a knot on one end, you can't get the first bead off until you have removed all the other beads. When an item is placed into a stack, it's known as _pushing_, and when an item is removed from a stack, it's called _popping_.

As the name implies, the stack segment of memory is, in fact, a stack data structure, which contains stack frames. The _ESP_ register is used to keep track of the address of the end of the stack, which is constantly changing as items are pushed onto the into and popped off of it. Since this is very dynamic behavior, it makes sense that the stack is also not of a fixed size. Opposite to the dynamic growth of the heap, as the stack changes in size, it grows upward in a visual listing of memory, toward lower memory addresses.

The _FILO_ nature of a stack might seem odd, but since the stack is used to store context, it's very useful. When a function is called, several things are pushed to the stack together in a _stack frame_. The _EBP_ register, sometimes called the _frame pointer_ (__FP__) or _local base pointer_ (__LB__), is used to reference local function variables in the current stack frame. Each stack frame contains the parameters to the function, its local variables, and two pointers that are necessary to put things back the way they were: The saved frame pointer (_SFP_) and the return address. The _SFP_ is used to restore _EBP_ to its previous value, and the _return address_ is used to restore _EIP_ to the next instruction found after the function call. This restores the functional context of the previous stack frame.

The following _stack_example.c_ code has two functions: _main()_ and _test_function()_.

__stack_example.c__

```c
void test_function(int a, int b, int c, int d)
{
    int flag;
    char buffer[10];

    flag = 31337;
    buffer[0] = 'A';
}

int main()
{
    test_function(1, 2, 3, 4);
}
```

This program first declares a test function that has four arguments, which are all declared as integers, _a_, _b_, _c_, and _d_. The local variables for the function include a single character called _flag_ and a 10-character buffer called _buffer_. The memory for these variables is in the stack segment, while the machine instructions for the function's code is stored in the text segment. After compiling the program, its inner workings can be examined with GDB. The following output shows the disassembled machine instructions for _main()_ and _test_function()_. The _main()_ function starts at _0x08048357_ and _test_function()_ starts at _0x08048344_. The first few instructions of each function (shown in bold below) set up the stack frame. These instructions are collectively called the _procedure prologue_ or _function prologue_. They save the frame pointer on the stack, and they save stack memory for the local function variables. Sometimes the function prologue will handle some stack alignment as well. The exact prologue instructions will vary greatly depending on the compiler and compiler options, but in general these instructions build the stack frame.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -g stack_example.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) disass main
Dump of assembler code for function main():
<strong><em>0x08048357 &lt;main+0&gt;:  push ebp</em></strong>
<strong><em>0x08048358 &lt;main+1&gt;:  mov ebp,esp</em></strong>
<strong><em>0x0804835a &lt;main+3&gt;:  sub esp,0x18</em></strong>
<strong><em>0x0804835d &lt;main+6&gt;:  and esp,0xfffffff0</em></strong>
<strong><em>0x08048360 &lt;main+9&gt;:  mov eax,0x0</em></strong>
<strong><em>0x08048365 &lt;main+14&gt;: sub esp,eax</em></strong>
0x08048367 &lt;main+16&gt;: mov DWORD PTR [esp+12],0x4
0x0804836f &lt;main+24&gt;: mov DWORD PTR [esp+8],0x3
0x08048377 &lt;main+32&gt;: mov DWORD PTR [esp+4],0x2
0x0804837f &lt;main+40&gt;: mov DWORD PTR [esp],0x1
0x08048386 &lt;main+47&gt;: call 0x8048344 &lt;test_function&gt;
0x0804838b &lt;main+52&gt;: leave
0x0804838c &lt;main+53&gt;: ret
End of assembler dump
(gdb) disass test_function()
Dump of assembler code for function test_function:
<strong><em>0x08048344 &lt;test_function+0&gt;:  push ebp</em></strong>
<strong><em>0x08048345 &lt;test_function+1&gt;:  mov ebp,esp</em></strong>
<strong><em>0x08048347 &lt;test_function+3&gt;:  sub esp,0x28</em></strong>
0x0804834a &lt;test_function+6&gt;:  mov DWORD PTR [ebp-12],0x7a69
0x08048351 &lt;test_function+13&gt;: mov BYTE PTR [ebp-40],0x41
0x08048355 &lt;test_function+17&gt;: leave
0x08048356 &lt;test_function+18&gt;: ret
End of assembler dump
(gdb) 
</pre>

When the program is run, the _main()_ function is called, which simply calls _test_function()_.

When the _test_function()_ is called from the _main()_ function, the various values are pushed to the stack to create the start of the stack frame as follows. When _test_function()_ is called, the function arguments are pushed onto the stack in reverse order (since it's _FILO_). The arguments for the function are _1_, _2_, _3_, and _4_, so the subsequent push instructions push _4_, _3_, _2_, and finally _1_ onto the stack. These values correspond to the variables __d__, __c__, __b__, and __a__ in the function. The instructions that put these values on the stack are shown in bold in the _main()_ function's disassembly below.

<pre style="color: white;">
(gdb) disass main
Dump of assembler code for function main:
0x08048357 &lt;main+0&gt;:  push ebp
0x08048358 &lt;main+1&gt;:  mov ebp,esp
0x0804835a &lt;main+3&gt;:  sub esp,0x18
0x0804835d &lt;main+6&gt;:  and esp,0xfffffff0
0x08048360 &lt;main+9&gt;:  mov eax,0x0
0x08048365 &lt;main+14&gt;: sub esp,eax
<strong><em>0x08048367 &lt;main+16&gt;: mov DWORD PTR [esp+12],0x4</em></strong>
<strong><em>0x0804836f &lt;main+24&gt;: mov DWORD PTR [esp+8],0x3</em></strong>
<strong><em>0x08048377 &lt;main+32&gt;: mov DWORD PTR [esp+4],0x2</em></strong>
<strong><em>0x0804837f &lt;main+40&gt;: mov DWORD PTR [esp],0x1</em></strong>
0x08048386 &lt;main+47&gt;: call 0x8048344 &lt;test_function&gt;
0x0804838b &lt;main+52&gt;: leave
0x0804838c &lt;main+53&gt;: ret
End of assembler dump
(gdb)
</pre>

Next, when the assembly call instruction is executed, the return address is pushed onto the stack and the execution flow jumps to the start of _test_function()_ at _0x08048344_. The return address value will be the location of the instruction following the current _EIP_, specifically, the value stored during step 3 of the previously mentioned execution loop. In this case, the return address would point to the leave instruction in _main()_ at _0x0804838b_.

The call instruction both stores the return address on the stack and jumps _EIP_ to the beginning of _test_function()_ , so _test_function()_'s procedure prologue instructions finish building the stack frame. In this step, the current value of _EBP_ is pushed to the stack. This value is called the saved frame pointer (__SFP__) and is later used to restore _EBP_ back to its original state.

The current value of _ESP_ is then copied into _EBP_ to set the new frame pointer. This frame pointer is used to reference the local variables of the function (_flag_ and _buffer_). Memory is saved for these variables by subtracting from _ESP_. In the end, the stack frame looks something like this:

<div align="left" width="100%">
<img src="Stack_Frame.png?raw=true" alt="Stack Frame" width="70%">
</div>

We can watch the stack frame construction on the stack using GDB. In the following output, a breakpoint at the beginning of _test_function()_. GDB will put the first breakpoint before the function arguments are pushed to the stack, and the second breakpoint after _test_function()_'s procedure prologue. When the program is run, execution stops at the breakpoint, where the register's _ESP_ (stack pointer), _EBP_ (frame pointer), and _EIP_ (execution pointer) are examined.

<pre style="color: white;">
(gdb) list main
4
5          flag = 31337;
6          buffer[0] = 'A';
7      }
8
9      int main() {
10         test_function(1, 2, 3, 4);
11     }
(gdb) break 10
Breakpoint 1 at 0x8048367: file stack_example.c, line 10.
(gdb) break test_function
Breakpoint 2 at 0x804834a: file stack_example.c, line 5.
(gdb) run
Starting program: /home/reader/booksrc/a.out

Breakpoint 1, main () at stack_example.c:10
10 test_function(1, 2, 3, 4);
(gdb) i r esp ebp eip
esp 0xbffff7f0 0xbffff7f0
ebp 0xbffff808 0xbffff808
eip 0x8048367 0x8048367 &lt;main+16&gt;
(gdb) x/5i $eip
0x8048367 &lt;main+16&gt;: mov DWORD PTR [esp+12],0x4
0x804836f &lt;main+24&gt;: mov DWORD PTR [esp+8],0x3
0x8048377 &lt;main+32&gt;: mov DWORD PTR [esp+4],0x2
0x804837f &lt;main+40&gt;: mov DWORD PTR [esp],0x1
0x8048386 &lt;main+47&gt;: call 0x8048344 &lt;test_function&gt;
(gdb) 
</pre>

This breakpoint is right before the stack frame for the _test_function()_ call is created. This means the bottom of this new stack frame is at the current value of _ESP_, _0xbffff7f0_. The next breakpoint is right after the procedure prologue for _test_function()_, so continuing will build the stack frame. The output below shows similar information at the second breakpoint. The local variables (_flag_ and _buffer_) are referenced relative to the frame pointer (_EBP_).

<pre style="color: white;">
(gdb) cont
Continuing.
Breakpoint 2, test_function (a=1, b=2, c=3, d=4) at stack_example.c:5
5 flag = 31337;
(gdb) i r esp ebp eip
esp 0xbffff7c0 0xbffff7c0
ebp 0xbffff7e8 0xbffff7e8
eip 0x804834a 0x804834a &lt;test_function+6&gt;
(gdb) disass test_function
Dump of assembler code for function test_function:
0x08048344 &lt;test_function+0&gt;:  push ebp
0x08048345 &lt;test_function+1&gt;:  mov ebp,esp
0x08048347 &lt;test_function+3&gt;:  sub esp,0x28
0x0804834a &lt;test_function+6&gt;:  mov DWORD PTR [ebp-12],0x7a69
0x08048351 &lt;test_function+13&gt;: mov BYTE PTR [ebp-40],0x41
0x08048355 &lt;test_function+17&gt;: leave
0x08048356 &lt;test_function+18&gt;: ret
End of assembler dump.
(gdb) print $ebp-12
$1 = (void *) 0xbffff7dc
(gdb) print $ebp-40
$2 = (void *) 0xbffff7c0
(gdb) x/16xw $esp
0xbffff7c0: <strong><em>[1]</em></strong>0x00000000 0x08049548 0xbffff7d8 0x08048249
0xbffff7d0: 0xb7f9f729 0xb7fd6ff4 0xbffff808 0x080483b9
0xbffff7e0: 0xb7fd6ff4 <strong><em>[2]</em></strong>0xbffff89c <strong><em>[3]</em></strong>0xbffff808 <strong><em>[4]</em></strong>0x0804838b
0xbffff7f0: <strong><em>[5]</em></strong>0x00000001 0x00000002 0x00000003 0x00000004
(gdb)
</pre>

The stack frame is shown on the stack at the end. The four arguments to the function can be seen at the bottom of the stack frame (__[5]__), with the return address found directly on top (__[4]__). Above that is the saved frame pointer of _0xbffff808_ (__[3]__), which is what _EBP_ was in the previous stack frame. The rest of the memory is saved for the local stack variables: __flag__ and __buffer__. Calculating their relative address to _EBP_ show their exact locations in the stack frame. Memory for the _flag_ variable is shown at __[2]__ and memory for the buffer variable is shown at __[1]__. The extra space in the stack frame is just padding.

After the execution finishes, the entire stack frame is popped off the stack, and the _EIP_ is set to the return address so the program can continue execution. If another function was called within the function, another stack frame would be pushed onto the stack, and so on. As each function ends, its stack frame is popped off of the stack so execution can be returned to the previous function. This behavior is the reason this segment of memory is organized in a _FILO_ data structure.

The various segments of memory are arranged in the order they were presented, from the lower memory addresses to the higher memory addresses. Since most people are familiar with seeing numbered lists that count downward, the smaller memory addresses are shown at the top. Some texts have this reversed, which can be very confusing; so for this book, smaller memory addresses are always shown at the top. Most debuggers also display memory in this style, with the smaller memory addresses at the top and the higher ones at the bottom.

Since the heap and the stack are both dynamic, they both grow in different directions toward each other. This minimizes wasted space, allowing the stack to be larger if the heap is small and vice versa.

<div align="left" width="100%">
<img src="Stack_Layout.png?raw=true" alt="Stack Layout" width="60%">
</div>