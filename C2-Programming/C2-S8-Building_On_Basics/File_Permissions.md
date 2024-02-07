# *__File Permissions__*

If the __O_CREAT__ flag is used in access mode for the _open()_ function, an additional argument is needed to define the file permissions of the newly created file. This argument uses bit flags defined in _sys/stat.h_, which can be combined with each other using _bitwise OR_ logic.

| Flag | Description |
| --- | --- |
| `S_IRUSR` | Give the file read permission for the user (owner). |
| `S_IWUSR`  | Give the file write permission for the user (owner).. |
| `S_IXUSR`  | Give the file execute permission for the user (owner). |
| `S_IRGRP`  | Give the file read permission for the group. |
| `S_IWGRP`  | Give the file write permission for the group. |
| `S_IXGRP`  | Give the file execute permission for the group. |
| `S_IROTH`  | Give the file read permission for other (anyone). |
| `S_IWOTH`  | Give the file write permission for other (anyone). |
| `S_IXOTH`  | Give the file execute permission for other (anyone). |

If you are already familiar with Unix file permissions, those flags should
make perfect sense to you. If they don’t make sense, here’s a crash course in
Unix file permissions.
Every file has an owner and a group. These values can be displayed using
`ls -l` and are shown below in the following output.

<pre style="color: white;">
reader@hacking:~/booksrc $ ls -l /etc/passwd simplenote*
-rw-r--r-- 1 root root 1424 2007-09-06 09:45 /etc/passwd
<strong><em>-rwxr-xr-x 1 reader reader 8457 2007-09-07 02:51 simplenote</em></strong>
-rw------- 1 reader reader 1872 2007-09-07 02:51 simplenote.c
reader@hacking:~/booksrc $
</pre>

For the _/etc/passwd_ file, the owner is __root__ and the group is also root. For
the other two simplenote files, the owner is __reader__ and the group is __users__.

Read, write, and execute permissions can be turned on and off for three different fields: user, group, and other. User permissions describe what the owner of the file can do (read, write, and/or execute), group permissions describe what users in that group can do, and other permissions describe what everyone else can do. These fields are also displayed in the front of the `ls -l` output. First, the user read/write/execute permissions are displayed, using __r__ for _read_, __w__ for _write_, __x__ for _execute_, and __-__ for _off_. The next three characters display the group permissions, and the last three characters are for the other permissions. In the output above, the simplenote program has all three user permissions turned on (shown in bold). Each permission corresponds to a bit flag; __read__ is _4_ (_100_ in binary), __write__ is _2_ (_010_ in binary), and __execute__ is _1_ (_001_ in binary). Since each value only contains unique bits, a _bitwise OR_ operation achieves the same result as adding these numbers together does. These values can be added together to define permissions for user, group, and other using the `chmod` command.

<pre style="color: white;">
reader@hacking:~/booksrc $ chmod 731 simplenote.c
reader@hacking:~/booksrc $ ls -l simplenote.c
-rwx-wx--x 1 reader reader 1826 2007-09-07 02:51 simplenote.c
reader@hacking:~/booksrc $ chmod ugo-wx simplenote.c
reader@hacking:~/booksrc $ ls -l simplenote.c
-r-------- 1 reader reader 1826 2007-09-07 02:51 simplenote.c
reader@hacking:~/booksrc $ chmod u+w simplenote.c
reader@hacking:~/booksrc $ ls -l simplenote.c
-rw------- 1 reader reader 1826 2007-09-07 02:51 simplenote.c
reader@hacking:~/booksrc $
</pre>

The first command (`chmod 731`) gives _read_, _write_, and _execute_ permissions to the user, since the first number is _7_ (_4 + 2 + 1_), _write_ and _execute_ permissions to group, since the second number is _3_ (_2 + 1_), and only execute permission to other, since the third number is _1_. Permissions can also be added or subtracted using chmod. In the next chmod command, the argument `ugo-wx` means _Subtract write and execute permissions from user, group, and other_. The final `chmod u+w` command gives write permission to user.

In the simplenote program, the _open()_ function uses `S_IRUSR | S_IWUSR` for its additional permission argument, which means the _/tmp/notes_ file should only have user read and write permission when it is created.

<pre style="color: white;">
reader@hacking:~/booksrc $ ls -l /tmp/notes
-rw------- 1 reader reader 36 2007-09-07 02:52 /tmp/notes
reader@hacking:~/booksrc $
</pre>