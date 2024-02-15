# *__Function Pointers__*

A _pointer_ simply contains a memory address and is given a data type that describes where it points. Usually, pointers are used for variables; however, they can also be used for functions. The _funcptr_example.c_ program demonstrates the use of function pointers.

__funcptr_example.c__

```c
#include <stdio.h>

int func_one() 
{
    printf("This is function one\n");
    return 1;
}

int func_two() 
{
    printf("This is function two\n");
    return 2;
}

int main() 
{
    int value;
    int (*function_ptr) ();

    function_ptr = func_one;
    printf("function_ptr is 0x%08x\n", function_ptr);
    value = function_ptr();
    printf("value returned was %d\n", value);

    function_ptr = func_two;
    printf("function_ptr is 0x%08x\n", function_ptr);
    value = function_ptr();
    printf("value returned was %d\n", value);
}
```

In this program, a function pointer aptly named __function_ptr__ is declared in _main()_. This pointer is then set to point at the function _func_one()_ and is called; then it is set again and used to call _func_two()_. The output below shows the compilation and execution of this source code. 

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc funcptr_example.c
reader@hacking:~/booksrc $ ./a.out
function_ptr is 0x08048374
This is function one
value returned was 1
function_ptr is 0x0804838d
This is function two
value returned was 2
reader@hacking:~/booksrc $
</pre>