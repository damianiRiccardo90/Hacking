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

