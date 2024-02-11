# *__User IDs__*

Every user on a Unix system has a unique _user ID number_. This user ID can
be displayed using the `id` command.

<pre style="color: white;">
reader@hacking:~/booksrc $ id reader
<>uid=999(reader) gid=999(reader)
groups=999(reader),4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),4
4(video),46(plugdev),104(scanner),112(netdev),113(lpadmin),115(powerdev),117(a
dmin)
reader@hacking:~/booksrc $ id matrix
uid=500(matrix) gid=500(matrix) groups=500(matrix)
reader@hacking:~/booksrc $ id root
uid=0(root) gid=0(root) groups=0(root)
reader@hacking:~/booksrc $
</pre>

The root user with user ID 0 is like the administrator account, which has full access to the system. The `su` command can be used to switch to a different user, and if this command is run as root, it can be done without a password. The `sudo` command allows a single command to be run as the root user. On the LiveCD, sudo has been configured so it can be executed without a password, for simplicity’s sake. These commands provide a simple method to quickly switch between users.

<pre style="color: white;">
reader@hacking:~/booksrc $ sudo su jose
jose@hacking:/home/reader/booksrc $ id
uid=501(jose) gid=501(jose) groups=501(jose)
jose@hacking:/home/reader/booksrc $
</pre>

As the user _jose_, the simplenote program will run as jose if it is executed, but it won’t have access to the _/tmp/notes_ file. This file is owned by the user reader, and it only allows read and write permission to its owner.

<pre style="color: white;">
jose@hacking:/home/reader/booksrc $ ls -l /tmp/notes
-rw------- 1 reader reader 36 2007-09-07 05:20 /tmp/notes
jose@hacking:/home/reader/booksrc $ ./simplenote "a note for jose"
[DEBUG] buffer @ 0x804a008: 'a note for jose'
[DEBUG] datafile @ 0x804a070: '/tmp/notes'
[!!] Fatal Error in main() while opening file: Permission denied
jose@hacking:/home/reader/booksrc $ cat /tmp/notes
cat: /tmp/notes: Permission denied
jose@hacking:/home/reader/booksrc $ exit
exit
reader@hacking:~/booksrc $
</pre>

This is fine if reader is the only user of the simplenote program; however, there are many times when multiple users need to be able to access certain portions of the same file. For example, the _/etc/passwd_ file contains account information for every user on the system, including each user's default login shell. The command `chsh` allows any user to change his or her own login shell. This program needs to be able to make changes to the _/etc/passwd_ file, but only on the line that pertains to the current user's account. The solution to this problem in Unix is the _set user ID (__setuid__)_ permission. This is an additional file permission bit that can be set using `chmod`. When a program with this flag is executed, it runs as the user ID of the file's owner.

<pre style="color: white;">
reader@hacking:~/booksrc $ which chsh
/usr/bin/chsh
reader@hacking:~/booksrc $ ls -l /usr/bin/chsh /etc/passwd
-rw-r--r-- 1 root root 1424 2007-09-06 21:05 /etc/passwd
-rw<strong><em>s</em></strong>r-xr-x 1 root root 23920 2006-12-19 20:35 /usr/bin/chsh
reader@hacking:~/booksrc $
</pre>

The `chsh` program has the __setuid__ flag set, which is indicated by an __s__ in the `ls` output above. Since this file is owned by root and has the _setuid_ permission set, the program will run as the root user when _any_ user runs this program.

The _/etc/passwd_ file that `chsh` writes to is also owned by root and only allows the owner to write to it. The program logic in `chsh` is designed to only allow writing to the line in _/etc/passwd_ that corresponds to the user running the program, even though the program is effectively running as root. This means that a running program has both a _real user ID_ and an _effective user ID_. These IDs can be retrieved using the functions _getuid()_ and _geteuid()_, respectively, as shown in _uid_demo.c_.

__uid_demo.c__

```c
#include <stdio.h>

int main() 
{
    printf("real uid: %d\n", getuid());
    printf("effective uid: %d\n", geteuid());
}
```

The results of compiling and executing _uid_demo.c_ are as follows.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o uid_demo uid_demo.c
reader@hacking:~/booksrc $ ls -l uid_demo
-rwxr-xr-x 1 reader reader 6825 2007-09-07 05:32 uid_demo
reader@hacking:~/booksrc $ ./uid_demo
real uid: 999
effective uid: 999
reader@hacking:~/booksrc $ sudo chown root:root ./uid_demo
reader@hacking:~/booksrc $ ls -l uid_demo
-rwxr-xr-x 1 root root 6825 2007-09-07 05:32 uid_demo
reader@hacking:~/booksrc $ ./uid_demo
real uid: 999
effective uid: 999
reader@hacking:~/booksrc $
</pre>

In the output for _uid_demo.c_, both user IDs are shown to be _999_ when _uid_demo_ is executed, since _999_ is the user ID for reader. Next, the `sudo` command is used with the `chown` command to change the owner and group of _uid_demo_ to __root__. The program can still be executed, since it has execute permission for _other_, and it shows that both user IDs remain _999_, since that’s still the ID of the user.

<pre style="color: white;">
reader@hacking:~/booksrc $ chmod u+s ./uid_demo
chmod: changing permissions of `./uid_demo': Operation not permitted
reader@hacking:~/booksrc $ sudo chmod u+s ./uid_demo
reader@hacking:~/booksrc $ ls -l uid_demo
-rwsr-xr-x 1 root root 6825 2007-09-07 05:32 uid_demo
reader@hacking:~/booksrc $ ./uid_demo
real uid: 999
effective uid: 0
reader@hacking:~/booksrc $
</pre>

Since the program is owned by _root_ now, `sudo` must be used to change file permissions on it. The `chmod u+s` command turns on the _setuid_ permission, which can be seen in the following `ls -l` output. Now when the user reader executes _uid_demo_, the _effective user ID_ is _0_ for _root_, which means the program can access files as _root_. This is how the `chsh` program is able to allow any user to change his or her login shell store in _/etc/passwd_.

The same technique can be used in a multiuser note-taking program. The next program will be a modification of the simplenote program; it will also record the _user ID_ of each note's original author. In addition, a new syntax for _#include_ will be introduced.

The _ec_malloc()_ and _fatal()_ functions have been useful in many of our programs. Rather than copy and paste these functions into each program, they can be put in a separate include file.

__hacking.h__

```c
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

In this new program, _hacking.h_, the function can just be included. In C, when the filename for a _#include_ is surrounded by __<__ and __>__, the compiler looks for this file in standard include paths, such as _/usr/include/_. If the filename is surrounded by quotes, the compiler looks in the current directory. 

Therefore, if _hacking.h_ is in the same directory as a program, it can be included with that program by typing `#include "hacking.h"`.

__notetaker.c__

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
//////////////////////////// NEW CODE ////////////////////////////
#include "hacking.h"
//////////////////////////// NEW CODE ////////////////////////////

void usage(char* prog_name, char* filename) 
{
    printf("Usage: %s <data to add to %s>\n", prog_name, filename);
    exit(0);
}

void fatal(char*);             // A function for fatal errors
void* ec_malloc(unsigned int); // An error-checked malloc() wrapper

int main(int argc, char* argv[]) 
{
    //////////////////////////// NEW CODE ////////////////////////////
    int userid, fd; // File descriptor
    //////////////////////////// NEW CODE ////////////////////////////
    char* buffer, *datafile;
    
    buffer = (char*) ec_malloc(100);
    datafile = (char*) ec_malloc(20);
    //////////////////////////// NEW CODE ////////////////////////////
    strcpy(datafile, "/var/notes");
    //////////////////////////// NEW CODE ////////////////////////////
    
    if (argc < 2)                 // If there aren't command-line arguments,
        usage(argv[0], datafile); // display usage message and exit.
    
    strcpy(buffer, argv[1]); // Copy into buffer.
    
    printf("[DEBUG] buffer @ %p: \'%s\'\n", buffer, buffer);
    printf("[DEBUG] datafile @ %p: \'%s\'\n", datafile, datafile);

// Opening the file
    fd = open(datafile, O_WRONLY|O_CREAT|O_APPEND, S_IRUSR|S_IWUSR);
    if (fd == -1)
        fatal("in main() while opening file");
    printf("[DEBUG] file descriptor is %d\n", fd);

    //////////////////////////// NEW CODE ////////////////////////////
    userid = getuid(); // Get the real user ID.
    //////////////////////////// NEW CODE ////////////////////////////

    //////////////////////////// NEW CODE ////////////////////////////
// Writing data
    if (write(fd, &userid, 4) == -1) // Write user ID before note data.
        fatal("in main() while writing userid to file");
    write(fd, "\n", 1); // Terminate line.

    if(write(fd, buffer, strlen(buffer)) == -1) // Write note.
        fatal("in main() while writing buffer to file");
    write(fd, "\n", 1); // Terminate line.
    //////////////////////////// NEW CODE ////////////////////////////

// Closing file
    if (close(fd) == -1)
        fatal("in main() while closing file");

    printf("Note has been saved.\n");
    free(buffer);
    free(datafile);
}
```

The output file has been changed from _/tmp/notes_ to _/var/notes_, so the data is now stored in a more permanent place. The _getuid()_ function is used to get the real user ID, which is written to the datafile on the line before the note’s line is written. Since the _write()_ function is expecting a pointer for its source, the _& operator_ is used on the integer value _userid_ to provide its address.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o notetaker notetaker.c
reader@hacking:~/booksrc $ sudo chown root:root ./notetaker
reader@hacking:~/booksrc $ sudo chmod u+s ./notetaker
reader@hacking:~/booksrc $ ls -l ./notetaker
-rwsr-xr-x 1 root root 9015 2007-09-07 05:48 ./notetaker
reader@hacking:~/booksrc $ ./notetaker "this is a test of multiuser notes"
[DEBUG] buffer @ 0x804a008: 'this is a test of multiuser notes'
[DEBUG] datafile @ 0x804a070: '/var/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ ls -l /var/notes
-rw------- 1 root reader 39 2007-09-07 05:49 /var/notes
reader@hacking:~/booksrc $
</pre>

In the preceding output, the notetaker program is compiled and changed to be owned by root, and the _setuid_ permission is set. Now when the program is executed, the program runs as the __root__ user, so the file _/var/notes_ is also owned by __root__ when it is created.

<pre style="color: white;">
reader@hacking:~/booksrc $ cat /var/notes
cat: /var/notes: Permission denied
reader@hacking:~/booksrc $ sudo cat /var/notes
?
this is a test of multiuser notes
reader@hacking:~/booksrc $ sudo hexdump -C /var/notes
00000000 <strong><em>e7 03 00 00</em></strong> 0a 74 68 69 73 20 69 73 20 61 20 74 |.....this is a t|
00000010 65 73 74 20 6f 66 20 6d 75 6c 74 69 75 73 65 72 |est of multiuser|
00000020 20 6e 6f 74 65 73 0a                            | notes.|
00000027
reader@hacking:~/booksrc $ pcalc 0x03e7
999 0x3e7 0y1111100111
reader@hacking:~/booksrc $
</pre>

The _/var/notes_ file contains the _user ID_ of reader (_999_) and the note. Because of little-endian architecture, the _4 bytes_ of the integer _999_ appear reversed in hexadecimal (shown in bold above). 

In order for a normal user to be able to read the note data, a corresponding _setuid root_ program is needed. The _notesearch.c_ program will read the note data and only display the notes written by that user ID. Additionally, an optional command-line argument can be supplied for a search string. When this is used, only notes matching the search string will be displayed.

__notesearch.c__

```c
#include <stdio.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
#include "hacking.h"

#define FILENAME "/var/notes"

int print_notes(int, int, char*); // Note printing function.
int find_user_note(int, int);     // Seek in file for a note for user.
int search_note(char*, char*);    // Search for keyword function.
void fatal(char*);                // Fatal error handler

int main(int argc, char* argv[]) 
{
    int userid, printing = 1, fd; // File descriptor
    char searchstring[100];

    if (argc > 1)                      // If there is an arg,
        strcpy(searchstring, argv[1]); // that is the search string;
    else // otherwise,
        searchstring[0] = 0;           // search string is empty.

    userid = getuid();
    fd = open(FILENAME, O_RDONLY); // Open the file for read-only access.
    if (fd == -1)
        fatal("in main() while opening file for reading");
    
    while(printing)
        printing = print_notes(fd, userid, searchstring);
    printf("-------[ end of note data ]-------\n");
    close(fd);
}

// A function to print the notes for a given uid that match
// an optional search string;
// returns 0 at end of file, 1 if there are still more notes.
int print_notes(int fd, int uid, char* searchstring) 
{
    int note_length;
    char byte = 0, note_buffer[100];

    note_length = find_user_note(fd, uid);
    if (note_length == -1) // If end of file reached,
        return 0; // return 0.

    read(fd, note_buffer, note_length); // Read note data.
    note_buffer[note_length] = 0; // Terminate the string.

    if (search_note(note_buffer, searchstring)) // If searchstring found,
        printf(note_buffer); // print the note.
    return 1;
}

// A function to find the next note for a given userID;
// returns -1 if the end of the file is reached;
// otherwise, it returns the length of the found note.
int find_user_note(int fd, int user_uid) 
{
    int note_uid = -1;
    unsigned char byte;
    int length;

    // Loop until a note for user_uid is found.
    while (note_uid != user_uid) 
    {
        if (read(fd, &note_uid, 4) != 4) // Read the uid data.
            return -1; // If 4 bytes aren't read, return end of file code.
        if (read(fd, &byte, 1) != 1) // Read the newline separator.
            return -1;

        byte = length = 0;
        // Figure out how many bytes to the end of line.
        while (byte != '\n') 
        {
            if (read(fd, &byte, 1) != 1) // Read a single byte.
                return -1; // If byte isn't read, return end of file code.
            length++;
        }
    }
    lseek(fd, length * -1, SEEK_CUR); // Rewind file reading by length bytes.

    printf("[DEBUG] found a %d byte note for user id %d\n", length, note_uid);
    return length;
}

// A function to search a note for a given keyword;
// returns 1 if a match is found, 0 if there is no match.
int search_note(char* note, char* keyword) 
{
    int i, keyword_length, match = 0;

    keyword_length = strlen(keyword);
    if (keyword_length == 0) // If there is no search string,
        return 1;            // always "match".

    // Iterate over bytes in note.
    for (i = 0; i < strlen(note); i++) 
    {
        if (note[i] == keyword[match]) // If byte matches keyword,
            match++; // get ready to check the next byte;
        else // otherwise,
        {
            if (note[i] == keyword[0]) // if that byte matches first keyword byte,
                match = 1; // start the match count at 1.
            else
                match = 0; // Otherwise it is zero.
        }
        if (match == keyword_length) // If there is a full match,
            return 1; // return matched.
    }
    return 0; // Return not matched.
}
```

Most of this code should make sense, but there are some new concepts. The filename is defined at the top instead of using heap memory. Also, the function __lseek()__ is used to rewind the read position in the file. The function call of `lseek(fd, length * -1, SEEK_CUR);` tells the program to move the read position forward from the current position in the file by _length * -1_ bytes. Since this turns out to be a negative number, the position is moved backward by __length__ bytes.

<pre style="color: white;">
reader@hacking:~/booksrc $ gcc -o notesearch notesearch.c
reader@hacking:~/booksrc $ sudo chown root:root ./notesearch
reader@hacking:~/booksrc $ sudo chmod u+s ./notesearch
reader@hacking:~/booksrc $ ./notesearch
[DEBUG] found a 34 byte note for user id 999
this is a test of multiuser notes
-------[ end of note data ]-------
reader@hacking:~/booksrc $
</pre>

When compiled and _setuid_ root, the notesearch program works as expected. But this is just a single user; what happens if a different user uses the notetaker and notesearch programs?

<pre style="color: white;">
reader@hacking:~/booksrc $ sudo su jose
jose@hacking:/home/reader/booksrc $ ./notetaker "This is a note for jose"
[DEBUG] buffer @ 0x804a008: 'This is a note for jose'
[DEBUG] datafile @ 0x804a070: '/var/notes'
[DEBUG] file descriptor is 3
Note has been saved.
jose@hacking:/home/reader/booksrc $ ./notesearch
[DEBUG] found a 24 byte note for user id 501
This is a note for jose
-------[ end of note data ]-------
jose@hacking:/home/reader/booksrc $
</pre>

When the user jose uses these programs, the _real user ID_ is _501_. This means that value is added to all notes written with notetaker, and only notes with a matching user ID will be displayed by the notesearch program.

<pre style="color: white;">
reader@hacking:~/booksrc $ ./notetaker "This is another note for the reader user"
[DEBUG] buffer @ 0x804a008: 'This is another note for the reader user'
[DEBUG] datafile @ 0x804a070: '/var/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ ./notesearch
[DEBUG] found a 34 byte note for user id 999
this is a test of multiuser notes
[DEBUG] found a 41 byte note for user id 999
This is another note for the reader user
-------[ end of note data ]-------
reader@hacking:~/booksrc $
</pre>

Similarly, all notes for the user reader have the user ID _999_ attached to them. Even though both the notetaker and notesearch programs are _suid_ root and have full read and write access to the _/var/notes_ datafile, the program logic in the notesearch program prevents the current user from viewing other users’ notes. This is very similar to how the _/etc/passwd_ file stores user information for all users, yet programs like `chsh` and `passwd` allow any user to change his own shell or password.