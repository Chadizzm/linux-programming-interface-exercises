Chapter 05: File I/O: Further Details
=====================================

Exercise 5-1
------------

**Question**

Modify the program in listing 5-3 to use standard file I/O system
calls (open() and lseek()) and the off_t data type.  Compile the
program with the _FILE_OFFSET_BITS macro set to 64, and test it to
show that a large file can be successfully created.

**Answer**

See large_file.c.  Just adding the macro seemed to do the trick.

Exercise 5-2
------------

**Question**
Write a program that opens and existing file for writing with the
O_APPEND flag, and then seeks to the beginning of the file before
writing some data.  Where does the data appear in the file?  Why?

**Answer**

The data is appended to the end of the file, rather than being
written to where one would expect.  This is the case because
the O_APPEND flag changes the write() system call to work
differently.  Instead of writing at the current offset, the OS
atomically seeks to the end of the file and performs the write.
Essentially, the offset value (pointer in file) doesn't matter
when opened with O_APPEND.

Exercise 5-3
------------

**Question**

This exercise is designed to demonstrate why the atomicity guaranteed
by opening a file with the O_APPEND flag is necessary.  Write a
program that takes up to three command-line arguments:

    $ atomic_append <filename> <num-bytes> [x]

This program should open the specified filename (creating it if
necessary) and append num-bytes bytes to the file by using write() to
write a byte at a time.  By default, the program should open the file
with the O_APPEND flag, but if a third command-line argument (x) is
supplied, the O_APPEND flag should be omitted, and instead, the
program should perform and lseek(fd, 0, SEEK_END) call before each
write().  Run two instances of this program at the same time without
the x argument to write 1 million bytes to the same file:

    $ atomic_append f1 1000000 & atomic_append f1 1000000

Repeat the same steps, writing to a different file, but this time
specifying the x argument:

    $ atomic_append f2 1000000 x & atomic_append f2 1000000 x

List the sizes of the files f1 and f2 using `ls -l` and explain the
difference.

**Answer**

The sizes were definitely different:

    -rw------- 1 posborne posborne 1272426 2012-01-15 21:31 test2.txt
    -rw------- 1 posborne posborne 2000000 2012-01-15 21:29 test.txt

Where test2.txt was run without O_APPEND.  test2.txt is short by the number
of times (or bytes as a result of times) that seeking to the end of the
file did not happen at the same time as the write (quite frequently).

Exercise 5-4
------------

**Question**

Implement dup() and dup2() using fcntl() and, where necessary,
close().  (You may ignore the fact that dup2() and fcntl() return
different errno values for some error cases.)  For dup2(), remember to
handle the special case where oldfd equals newfd.  In this case, you
should check whether oldfd is valid, which can be done by, for
example, checking if fcntl(oldfd, F_GETFL) succeeds.  If oldfd is not
valid, then the function should return -1 with errrno set to EBADF.

**Answer**

See dup.c.  Note that in the test app, the files share the same
offset (as shown by the reads).

Exercise 5-5
------------

**Question**

Write a program to verify that duplicated file descriptors share a
file offset value and open file status flags.

**Answer**

See the main entry-point for dup.c

Exercise 5-6
------------

**Question**

After each of the calls to write() in the following code, explain
what the content of the output file would be, and why:

    fd1 = open(file, O_RDWR | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);
    fd2 = dup(fd1);
    fd3 = open(file, O_RDWR);
    write(fd1, "Hello,", 6);
    write(fd2, " world", 6);
    lseek(fd2, 0, SEEK_SET);
    write(fd1, "HELLO,", 6);
    write(fd3, "Gidday", 6);

**Answer**

    >>> fd1 = open(file, O_RDWR | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);

Attempt to create a new file descriptor for the file "file" for read/write
access.  If it does not exist, create is with user read/write permissions.
If the file already exists, the contents are truncated (no contents and
offset set at 0).  On failure, -1 will be returned.

    >>> fd2 = dup(fd1);

Create a duplicate of the file descriptor fd1.  fd2, on success, will
share the same offset/flags as fd1.

    >>> fd3 = open(file, O_RDWR);

Create a new file descriptor and open with read/write permissions.  If
the file does not exists it will not be created.  The contents will
not be affected by the file pointer will be at the beginning of the file.

    >>> write(fd1, "Hello,", 6);

From the current position in the file, write (or overwrite) the 6 bytes
"Hello," to the file.  The number of bytes actually written will be
returned (or -1 on failure).  The operation may act slightly differently
depending on the flags.  In the current context, "Hello," will be
written to "file", starting at the beginning of the file.  The offset will
be modified on fd1 and fd2 but not on fd3.

    >>> write(fd2, " world", 6);

Like the previous, affecting the offset in fd1 and fd2 but not affecting
fd3.  File now contains "Hello, world".

    >>> lseek(fd2, 0, SEEK_SET);

Sets fd1 and fd2 file offset to the beggining of the file.

    >>> write(fd1, "HELLO,", 6);

File now contains "HELLO, world"

    >>> write(fd3, "Gidday", 6);

Since the offset is not shared, this results in the following file
contents "Gidday world"

Exercise 5-7
------------

**Question**

Implemented readv() and writev() using read() and write(), and
suitable functions from the malloc package (Section 7.1.2).

**Answer**

See vector_fileio.c
