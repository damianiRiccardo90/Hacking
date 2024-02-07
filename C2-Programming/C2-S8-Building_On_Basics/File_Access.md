# *__File Access__*

There are two primary ways to access files in C: File descriptors and filestreams. _File descriptors_ use a set of low-level I/O functions, and _filestreams_ are a higher-level form of buffered I/O that is build on the lower-level functions. Some consider the filestream functions easier to program with; however, file descriptors are more direct. In this book, the focus will be on the low-level I/O functions that use file descriptors.

The bar code on the back of this book represents a number. Because this number is unique among the other books in a bookstore, the cashier can scan the number at checkout and use it to reference information about this book in the store's database. Similarly, a file descriptor is a number that is used to reference open files. Four common functions that use file descriptors are _open()_, _close()_, _read()_, and _write()_. All of these functions will return _-1_ if there is an error. The _open()_ function opens a file for reading and/or writing and returns a file descriptor. The returned file descriptor is just an integer value, but it is unique among open files. The file descriptor is passed as an argument to the other functions like a pointer to the opened file. 

For the _close()_ function, the file descriptor is the only argument. The _read()_ and _write()_ functions' arguments are the file descriptor, a pointer to the data to read or write, and the number of bytes to read or write from that location. The arguments to the _open()_ function are a pointer to the filename to open and a series of predefined flags that specify the access mode. These flags and their usage will be explained in depth later, but for now let's take a look at a simple note-taking program that uses file descriptors, _simplenote.c_. This program accepts a note as a command-line argument and then adds it to the end of the file _/tmp/notes_. This program uses several functions, including a familiar looking error-checked heap memory allocation function. Other functions are used to display a usage message and to handle fatal errors. The _usage()_ function is simply defined before _main()_, so it doesn't need a function prototype.

__simplenote.c__

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>

void usage(char* prog_name, char* filename) 
{
    printf("Usage: %s <data to add to %s>\n", prog_name, filename);
    exit(0);
}

void fatal(char*); // A function for fatal errors
void* ec_malloc(unsigned int); // An error-checked malloc() wrapper

int main(int argc, char* argv[]) 
{
    int fd; // file descriptor
    char* buffer, *datafile;

    buffer = (char*) ec_malloc(100);
    datafile = (char*) ec_malloc(20);
    strcpy(datafile, "/tmp/notes");

    if (argc < 2) // If there aren't command-line arguments,
        usage(argv[0], datafile); // display usage message and exit.

    strcpy(buffer, argv[1]); // Copy into buffer.

    printf("[DEBUG] buffer @ %p: \'%s\'\n", buffer, buffer);
    printf("[DEBUG] datafile @ %p: \'%s\'\n", datafile, datafile);

    strncat(buffer, "\n", 1); // Add a newline on the end.

// Opening file
    fd = open(datafile, O_WRONLY|O_CREAT|O_APPEND, S_IRUSR|S_IWUSR);
    if (fd == -1)
        fatal("in main() while opening file");
    printf("[DEBUG] file descriptor is %d\n", fd);
// Writing data
    if (write(fd, buffer, strlen(buffer)) == -1)
        fatal("in main() while writing buffer to file");
// Closing file
    if (close(fd) == -1)
        fatal("in main() while closing file");
    
    printf("Note has been saved.\n");
    free(buffer);
    free(datafile);
}

// A function to display an error message and then exit
void fatal(char* message) 
{
    char error_message[100];

    strcpy(error_message, "[!!] Fatal Error ");
    strncat(error_message, message, 83);
    perror(error_message);
    exit(-1);
}

// An error-checked malloc() wrapper function
void* ec_malloc(unsigned int size) 
{
    void* ptr;
    ptr = malloc(size);
    if (ptr == NULL)
        fatal("in ec_malloc() on memory allocation");
    return ptr;
}
```

Besides the strange-looking flags used in the _open()_ function, most of this code should be readable. There are also a few standard functions that we haven't used before. The _strlen()_ function accepts a string and returns its length. It's used in combination with the _write()_ function ,since it needs to know how many bytes to write. The _perror()_ function is short for _print error_ and is used in _fatal()_ to print an additional error message (if it exists) before exiting.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o simplenote simplenote.c
reader@hacking:~/booksrc $ ./simplenote
Usage: ./simplenote &lt;data to add to /tmp/notes&gt;
reader@hacking:~/booksrc $ ./simplenote "this is a test note"
[DEBUG] buffer @ 0x804a008: 'this is a test note'
[DEBUG] datafile @ 0x804a070: '/tmp/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ cat /tmp/notes
this is a test note
reader@hacking:~/booksrc $ ./simplenote "great, it works"
[DEBUG] buffer @ 0x804a008: 'great, it works'
[DEBUG] datafile @ 0x804a070: '/tmp/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ cat /tmp/notes
this is a test note
great, it works
reader@hacking:~/booksrc $
</pre>

The output of the program's execution is pretty self-explanatory, but there are some things about the source code that need further explanation., The files _fcntl.h_ and _sys/stat.h_ had to be included, since those files define the flags used with the _open()_ function. The first set of flags is found in _fcntl.h_ and is used to set the access mode. The access mode must use at least one the following three flags:

| Flag | Description |
| --- | --- |
| `O_RDONLY` | Open file for read-only access. |
| `O_WRONLY` | Open file for write-only access. |
| `O_RDWR`   | Open file for both read and write access. |

These flags can be combined with several other optional flags using the _bitwise OR operator_. A few of the more common and useful of these flags are as follows:

| Flag | Description |
| --- | --- |
| `O_APPEND` | Write data at the end of the file. |
| `O_TRUNC`  | If the file already exists, truncate the file to 0 length. |
| `O_CREAT`  | Create the file if it doesn't exist. |

Bitwise operations combine bits using standard logic gates such as _OR_ and _AND_. When two bits enter an _OR_ gate, the result is _1_ if either the first bit _or_ the second bit is _1_. If two bits enter an _AND_ gate, the result is _1_ only if both the first bit _and_ the second bit are _1_. Full 32-bit values can use these bitwise operators to perform logic operations on each corresponding bit. The source code of _bitwise.c_ and the program output demonstrate these bitwise operations.

__bitwise.c__

```c
#include <stdio.h>

int main() 
{
    int i, bit_a, bit_b;
    printf("bitwise OR operator |\n");
    for (i = 0; i < 4; i++) 
    {
        bit_a = (i & 2) / 2; // Get the second bit.
        bit_b = (i & 1);     // Get the first bit.
        printf("%d | %d = %d\n", bit_a, bit_b, bit_a | bit_b);
    }
    printf("\nbitwise AND operator &\n");
    for (i = 0; i < 4; i++) 
    {
        bit_a = (i & 2) / 2; // Get the second bit.
        bit_b = (i & 1);     // Get the first bit.
        printf("%d & %d = %d\n", bit_a, bit_b, bit_a & bit_b);
    }
}
```

The result of compiling and executing _bitwise.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc bitwise.c
reader@hacking:~/booksrc $ ./a.out
bitwise OR operator |
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1

bitwise AND operator &
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1
reader@hacking:~/booksrc $
</pre>

The flags used for the _open()_ function have values that correspond to single bits, This way, flags can be combined using _OR_ logic without destroying any information. The _fcntl_flags.c_ program and its output explore some of the flag values defined by _fcntl.h_ and how they combine with each other.

__fcntl.c__

```c
#include <stdio.h>
#include <fcntl.h>

void display_flags(char*, unsigned int);
void binary_print(unsigned int);

int main(int argc, char* argv[]) 
{
    display_flags("O_RDONLY\t\t", O_RDONLY);
    display_flags("O_WRONLY\t\t", O_WRONLY);
    display_flags("O_RDWR\t\t\t", O_RDWR);
    printf("\n");
    display_flags("O_APPEND\t\t", O_APPEND);
    display_flags("O_TRUNC\t\t\t", O_TRUNC);
    display_flags("O_CREAT\t\t\t", O_CREAT);
    printf("\n");
    display_flags("O_WRONLY|O_APPEND|O_CREAT", O_WRONLY|O_APPEND|O_CREAT);
}

void display_flags(char* label, unsigned int value) 
{
    printf("%s\t: %d\t:", label, value);
    binary_print(value);
    printf("\n");
}

void binary_print(unsigned int value) 
{
    unsigned int mask = 0xff000000;       // Start with a mask for the highest byte.
    unsigned int shift = 256 * 256 * 256; // Start with a shift for the highest byte.
    unsigned int byte, byte_iterator, bit_iterator;
    
    for (byte_iterator = 0; byte_iterator < 4; byte_iterator++) 
    {
        byte = (value & mask) / shift; // Isolate each byte.
        printf(" ");
        // Print the byte's bits.
        for (bit_iterator = 0; bit_iterator < 8; bit_iterator++) 
        {
            if (byte & 0x80) // If the highest bit in the byte isn't 0,
                printf("1"); // print a 1.
            else
                printf("0"); // Otherwise, print a 0.
            byte *= 2;       // Move all the bits to the left by 1.
        }
        mask /= 256;         // Move the bits in mask right by 8.
        shift /= 256;        // Move the bits in shift right by 8.
    }
}
```

The results of compiling and executing _fcntl_flags.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc fcntl_flags.c
reader@hacking:~/booksrc $ ./a.out
O_RDONLY                  : 0    : 00000000 00000000 00000000 00000000
O_WRONLY                  : 1    : 00000000 00000000 00000000 00000001
O_RDWR                    : 2    : 00000000 00000000 00000000 00000010
O_APPEND                  : 1024 : 00000000 00000000 00000100 00000000
O_TRUNC                   : 512  : 00000000 00000000 00000010 00000000
O_CREAT                   : 64   : 00000000 00000000 00000000 01000000
O_WRONLY|O_APPEND|O_CREAT : 1089 : 00000000 00000000 00000100 01000001
$
</pre>

Using bit flags in combination with bitwise logic is an efficient and commonly used technique. As long as each flag is a number that only has unique bits turned on, the effect of doing a _bitwise OR_ on these values is the same as adding them. In _fcntl_flags.c_, _1 + 1024 + 64 = 1089_. This technique only works
when all the bits are unique, though.

