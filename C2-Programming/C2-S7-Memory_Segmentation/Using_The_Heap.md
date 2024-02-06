# *__Using the Heap__*

Using the other memory segments is simply a matter of how you declare variables. However, using the heap requires a bit more effort. As previously demonstrated, allocating memory on the heap is done using the __malloc()__ function. This function accepts a size as its only argument and reserves that much space in the heap segment, returning the address to the start of this memory as a void pointer.- If the _malloc()_ function can't allocate memory for some reason, it will simply return a __NULL__ pointer with a value of _0_.

The corresponding deallocation function is __free()__. This function accepts a pointer as its only argument and frees that memory space on the heap so it can be used again later. The relatively simple functions are demonstrated in _heap_example.c_.

__heap_example.c__

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char* argv[]) 
{
    char* char_ptr; // A char pointer
    int* int_ptr; // An integer pointer
    int mem_size;
    
    if (argc < 2)      // If there aren't command-line arguments,
        mem_size = 50; // use 50 as the default value.
    else
        mem_size = atoi(argv[1]);

    printf("\t[+] allocating %d bytes of memory on the heap for char_ptr\n", mem_size);
    char_ptr = (char*) malloc(mem_size); // Allocating heap memory

    // Error checking, in case malloc() fails
    if(char_ptr == NULL) 
    {
        fprintf(stderr, "Error: could not allocate heap memory.\n");
        exit(-1);
    }

    strcpy(char_ptr, "This is memory is located on the heap.");
    printf("char_ptr (%p) --> '%s'\n", char_ptr, char_ptr);

    printf("\t[+] allocating 12 bytes of memory on the heap for int_ptr\n");
    int_ptr = (int*) malloc(12); // Allocated heap memory again

    // Error checking, in case malloc() fails
    if(int_ptr == NULL) 
    {
        fprintf(stderr, "Error: could not allocate heap memory.\n");
        exit(-1);
    }

    *int_ptr = 31337; // Put the value of 31337 where int_ptr is pointing.
    printf("int_ptr (%p) --> %d\n", int_ptr, *int_ptr);

    printf("\t[-] freeing char_ptr's heap memory...\n");
    free(char_ptr); // Freeing heap memory

    printf("\t[+] allocating another 15 bytes for char_ptr\n");
    char_ptr = (char*) malloc(15); // Allocating more heap memory

    // Error checking, in case malloc() fails
    if(char_ptr == NULL) 
    {
        fprintf(stderr, "Error: could not allocate heap memory.\n");
        exit(-1);
    }

    strcpy(char_ptr, "new memory");
    printf("char_ptr (%p) --> '%s'\n", char_ptr, char_ptr);
    
    printf("\t[-] freeing int_ptr's heap memory...\n");
    free(int_ptr); // Freeing heap memory

    printf("\t[-] freeing char_ptr's heap memory...\n");
    free(char_ptr); // Freeing the other block of heap memory
}
```

This program accepts a command-line argument for the size of the first memory allocation, with a default value of _50_. Then it uses the _malloc()_ and _free()_ functions to allocate and deallocate memory on the heap. There are plenty of _printf()_ statements to debug what is actually happening when the program is executed. Since _malloc()_ doesn't know what type of memory it's allocating, it returns a void pointer to the newly allocated heap memory, which must be typecast into the appropriate type. After every _malloc()_ call, there is an error-checking block that checks whether or not the allocation failed. If the allocation fails and the pointer is _NULL_, _fprintf()_ is used to print an error message to standard error and the program exits. The _fprintf()_ is used to print an error message to standard error and the program exits. The _fprintf()_ function is very similar to _printf()_; however, its first argument is __stderr__, which is a standard filestream meant for displaying errors. This function will be explained more later, but for now, it's just used as a way to properly display an error. The rest of the program is pretty straightforward.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o heap_example heap_example.c
reader@hacking:~/booksrc $ ./heap_example
        [+] allocating 50 bytes of memory on the heap for char_ptr
char_ptr (0x804a008) --> 'This is memory is located on the heap.'
        [+] allocating 12 bytes of memory on the heap for int_ptr
int_ptr (0x804a040) --> 31337
        [-] freeing char_ptr's heap memory...
        [+] allocating another 15 bytes for char_ptr
char_ptr (0x804a050) --> 'new memory'
        [-] freeing int_ptr's heap memory...
        [-] freeing char_ptr's heap memory...
reader@hacking:~/booksrc $ 
</pre>

In the preceding output, notice that each block of memory has an incrementally higher memory address in the heap. Even though the firs 50 bytes were deallocated, when 15 more bytes are requested