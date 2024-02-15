# *__Structs__*

Sometimes there are multiple variables that should be grouped together and treated like one. In C, __structs__ are variables that can contain many other variables. Structs are often used by various system functions and libraries, so understanding how to use structs is a prerequisite to using these functions.

A simple example will suffice for now. When dealing with many time functions,
these functions use a time struct called __tm__, which is defined in _/usr/include/
time.h_. The struct’s definition is as follows.

```c
struct tm {
int     tm_sec;     /* seconds */
int     tm_min;     /* minutes */
int     tm_hour;    /* hours */
int     tm_mday;    /* day of the month */
int     tm_mon;     /* month */
int     tm_year;    /* year */
int     tm_wday;    /* day of the week */
int     tm_yday;    /* day in the year */
int     tm_isdst;   /* daylight saving time */
};
```

After this struct is defined, `struct tm` becomes a usable variable type, which can be used to declare variables and pointers with the data type of the tm struct. The _time_example.c_ program demonstrates this. When _time.h_ is included, the __tm__ struct is defined, which is later used to declare the __current_time__ and __time_ptr__ variables.

__time_example.c__

```c
#include <stdio.h>
#include <time.h>

int main() 
{
    long int seconds_since_epoch;
    struct tm current_time, *time_ptr;
    int hour, minute, second, day, month, year;

    seconds_since_epoch = time(0); // Pass time a null pointer as argument.
    printf("time() - seconds since epoch: %ld\n", seconds_since_epoch);

    time_ptr = &current_time; // Set time_ptr to the address of
                              // the current_time struct.
    localtime_r(&seconds_since_epoch, time_ptr);

    // Three different ways to access struct elements:
    hour = current_time.tm_hour; // Direct access
    minute = time_ptr->tm_min; // Access via pointer
    second = *((int*) time_ptr); // Hacky pointer access

    printf("Current time is: %02d:%02d:%02d\n", hour, minute, second);
}
```

The _time()_ function will return the number of seconds since January 1, 1970. Time on Unix systems is kept relative to this rather arbitrary point in time, which is also known as the __epoch__. The _localtime_r()_ function expects two pointers as arguments: one to the number of seconds since epoch and the other to a __tm__ struct. The pointer _time_ptr_ has already been set to the address of current_time, an empty __tm__ struct. The address-of operator is used to provide a pointer to __seconds_since_epoch__ for the other     argument to _localtime_r()_, which fills the elements of the __tm__ struct. The elements of structs can be accessed in three different ways; the first two are the proper ways to access struct elements, and the third is a hacked solution. If a struct variable is used, its elements can be accessed by adding the elements’ names to the end of the variable name with a period. Therefore, `current_time.tm_hour` will access just the __tm_hour__ element of the __tm__ struct called __current_time__. Pointers to structs are often used, since it is much more efficient to pass a four-byte pointer than an entire data structure. Struct pointers are so common that C has a built-in method to access struct elements from a struct pointer without needing to dereference the pointer. When using a struct pointer like __time_ptr__, struct elements can be similarly accessed by the struct element’s name, but using a series of characters that looks like an arrow pointing right. Therefore, `time_ptr->tm_min` will access the __tm_min__ element of the __tm__ struct that is pointed to by __time_ptr__. The seconds could be accessed via either of these proper methods, using the __tm_sec__ element or the __tm__ struct, but a third method is used. Can you figure out how this third method works?

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc time_example.c
reader@hacking:~/booksrc $ ./a.out
time() - seconds since epoch: 1189311588
Current time is: 04:19:48
reader@hacking:~/booksrc $ ./a.out
time() - seconds since epoch: 1189311600
Current time is: 04:20:00
reader@hacking:~/booksrc $
</pre>

The program works as expected, but how are the seconds being accessed in the __tm__ struct? Remember that in the end, it’s all just memory. Since __tm_sec__ is defined at the beginning of the __tm__ struct, that integer value is also found at the beginning. In the line `second = *((int *) time_ptr)`, the variable __time_ptr__ is typecast from a __tm__ struct pointer to an integer pointer. Then this typecast pointer is dereferenced, returning the data at the pointer’s address. Since the address to the __tm__ struct also points to the first element of this struct, this will retrieve the integer value for __tm_sec__ in the struct. The following addition to the _time_example.c_ code (_time_example2.c_) also dumps the bytes of the __current_time__. This shows that the elements of tm struct are right next to each other in memory. The elements further down in the struct can also be directly accessed with pointers by simply adding to the address of the pointer.

__time_example2.c__

```c
#include <stdio.h>
#include <time.h>

void dump_time_struct_bytes(struct tm* time_ptr, int size) 
{
    int i;
    unsigned char* raw_ptr;

    printf("bytes of struct located at 0x%08x\n", time_ptr);
    raw_ptr = (unsigned char*) time_ptr;
    for (i = 0; i < size; i++)
    {
        printf("%02x ", raw_ptr[i]);
        if (i % 16 == 15) // Print a newline every 16 bytes.
            printf("\n");
    }
    printf("\n");
}

int main() 
{
    long int seconds_since_epoch;
    struct tm current_time, *time_ptr;
    int hour, minute, second, i, *int_ptr;
    
    seconds_since_epoch = time(0); // Pass time a null pointer as argument.
    printf("time() - seconds since epoch: %ld\n", seconds_since_epoch);

    time_ptr = &current_time; // Set time_ptr to the address of
                              // the current_time struct.
    localtime_r(&seconds_since_epoch, time_ptr);

    // Three different ways to access struct elements:
    hour = current_time.tm_hour; // Direct access
    minute = time_ptr->tm_min;   // Access via pointer
    second = *((int*) time_ptr); // Hacky pointer access

    printf("Current time is: %02d:%02d:%02d\n", hour, minute, second);

    dump_time_struct_bytes(time_ptr, sizeof(struct tm));

    minute = hour = 0; // Clear out minute and hour.
    int_ptr = (int*) time_ptr;
    for (i = 0; i < 3; i++) 
    {
        printf("int_ptr @ 0x%08x : %d\n", int_ptr, *int_ptr);
        int_ptr++; // Adding 1 to int_ptr adds 4 to the address,
    }              // since an int is 4 bytes in size.
}
```

The results of compiling and executing _time_example2.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -g time_example2.c
reader@hacking:~/booksrc $ ./a.out
time() - seconds since epoch: 1189311744
Current time is: 04:22:24
bytes of struct located at 0xbffff7f0
18 00 00 00 16 00 00 00 04 00 00 00 09 00 00 00
08 00 00 00 6b 00 00 00 00 00 00 00 fb 00 00 00
00 00 00 00 00 00 00 00 28 a0 04 08
int_ptr @ 0xbffff7f0 : 24
int_ptr @ 0xbffff7f4 : 22
int_ptr @ 0xbffff7f8 : 4
reader@hacking:~/booksrc $
</pre>

While struct memory can be accessed this way, assumptions are made about the type of variables in the struct and the lack of any padding between variables. Since the data types of a struct’s elements are also stored in the struct, using proper methods to access struct elements is much easier.