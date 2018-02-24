---
layout: post
title: Linux File
key: 20160312
tags: Linux File C I/O
---

> Almost everything in Posix is handled through a file descriptor

File descriptors: These are small integers that you can use to access open files or devices.

Programs can use disk files, serial ports, printers and other devices in exactly the same way as they would use a file: open, close, read, write and ioctl.

![file descriptor](https://upload.wikimedia.org/wikipedia/commons/thumb/f/f8/File_table_and_inode_table.svg/460px-File_table_and_inode_table.svg.png)

> Each Unix process (except perhaps a daemon) should expect to have three standard POSIX file descriptors, corresponding to the three standard streams. *The file descriptors that are automatically opened, however, already allow us to create some simple programs using write.*

|Integer value|feilds|
|---|---|
|0|standard input|
|1|standard output|
|2|standard error|


### write

```
#include <unistd.h>
size_t write(int fildes, const void *buf, size_t nbytes);
```

### read

```
#include <unistd.h>
size_t read(int fildes, void *buf, size_t nbytes);
```

### open

To create a new file descriptor we need to use the open system call.

```
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
int open(const char *path, int oflags);
int open(const char *path, int oflags, mode_t mode);
```

>Note Strictly speaking, we don't need to include sys/types.h and sys/stat.h to use open on POSIX systems, but they may be necessary on some UNIX systems.

> If two programs have a file open at the same time, they maintain distinct file descriptors. If they both write to the file, they will continue to write where they left off. Their data isn't interleaved, but one will overwrite the other. Each keeps its own idea of how far into the file (the offset) it has read or written.

> The name of the file or device to be opened is passed as a parameter, path, and the oflags parameter is used to specify actions to be taken on opening the file.

The oflags are specified as a bitwise OR of a mandatory file access mode and other optional modes. The open
call must specify one of the following file access modes:


kernal use three different types of data structure to represent opened files

|Mode|Description|
|---|---|
|O_RDONLY|Open for read−only|
|O_WRONLY|Open for write−only|
|O_RDWR|Open for reading and writing|

The call may also include a combination (bitwise OR) of the following optional modes in the oflags
parameter:

|Mode|Description|
|---|---|
|O_APPEND|Place written data at the end of the file.|
|O_TRUNC |Set the length of the file to zero, discarding existing contents.|
|O_CREAT|Creates the file, if necessary, with permissions given in mode.|
|O_EXCL|Used with O_CREAT, ensures that the caller creates the file. The open is atomic, i.e. it's performed with just one function call. This protects against two programs creating the file at the same time. If the file already exists, open will fail.|

open returns the new file descriptor (always a non−negative integer) if successful, or −1 if it fails, when open also sets the global variable errno to indicate the reason for the failure. The new file descriptor is always the lowest numbered unused descriptor, a feature that can be quite useful in some circumstances.

### Initial Permissions

When we create a file using the O_CREAT flag with open, we must use the three parameter form. mode, the
third parameter, is made from a bitwise OR of the flags defined in the header file sys/stat.h.

|Mode|Permission|
|---|---|
|S_IRUSR| Read permission, owner.|
|S_IWUSR| Write permission, owner.|
|S_IXUSR| Execute permission, owner.|
|S_IRGRP| Read permission, group.|
|S_IWGRP| Write permission, group.|
|S_IXGRP| Execute permission, group.|
|S_IROTH| Read permission, others.|
|S_IWOTH| Write permission, others.|
|S_IXOTH| Execute permission, others.|

```
open ("myfile", O_CREAT, S_IRUSR|S_IXOTH);
```

string must be “”, char ''


### close

```
#include <unistd.h>
int close(int fildes);
```
It returns 0 if successful and −1 on error.


### standard IO

With that said, some things in Posix, and in particular, in Linux, are definitely not files.

The most obvious ones are signals. They are handled asynchronously to the program's execution, and therefor cannot take on a file interface. For that purpose, pselect and one of its better alternatives were invented.

Things more subtly not files are thread synchronization constructs (mutexs, semaphores, etc.). Some attempt have been made to make those available as file descriptors as well (see signalfd and eventfd), but those hardly caught on. I believe that this is due, in large part, for them having a vastly different performance profiles than the ususal way of handling them.
