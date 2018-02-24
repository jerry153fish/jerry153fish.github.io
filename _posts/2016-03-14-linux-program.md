---
layout: post
title: Linux Program Notes 
key: 20160314
tags: Linux Program Note
---

Applications under Linux / UNIX are represented by two special types of file: executables and scripts. Executable
files are programs that can be run directly by the computer and correspond to DOS .exe files. Scripts are
collections of instructions for another program, an interpreter, to follow. These correspond to DOS .bat files,
or interpreted BASIC programs.

> UNIX uses the : character to separate entries in the PATH variable windows ;

### Program location

Programs supplied by the system for general use, including program development, are found in /usr/bin. Programs added by system administrators for a specific host computer or local network are found in /usr/local/bin. Administrators favor /usr/local, as it keeps vendor supplied files and later additions separate from the programs supplied by the system. Keeping /usr organized in this way may help when the time comes to upgrade the operating system, since only /usr/local need be preserved. We recommend that you compile your programs to run and access required files from the /usr/local hierarchy.

### Header files

in /usr/include/sys and /usr/include/linux.

### Libaries files

Standard system libraries are usually stored in /lib and /usr/lib. A library name always starts with lib. Then follows the part indicating what library this is (like c for the C
library, or m for the mathematical library). The last part of the name starts with a dot ., and specifies the type
of the library:

- *.a for traditional, static libraries
- *.so and .sa for shared libraries (See below.)

```
$ cc −o fred fred.c −lm
```

The −lm (no space between the l and the m) is shorthand (Shorthand is much valued in UNIX circles.) for the
library called libm.a in one of the standard library directories (in this case /usr/lib). An additional advantage of
the −lm notation is that the compiler will automatically choose the shared library when it exists.
Although libraries usually are found in standard places in the same way as header files, we can add to the
search directories by using the −L (uppercase letter) flag to the compiler. For example,

```
$ cc −o x11fred −L/usr/openwin/lib x11fred.c −lX11
```

will compile and link a program called x11fred using the version of the library libX11 found in the directory
/usr/openwin/lib.

### shared libraries

shared libraries are similar to dynamic−link libraries used under Microsoft Windows. The .so
libraries correspond to .DLL files and are required at run time, while the .sa libraries are similar to .LIB files
that get included in the program executable.

### /dev

Device files in /dev are all used in the same way; they can all be opened, read, written and closed.

| operation | description|
|---|---|
| open| Open a file or device.|
| read| Read from an open file or device.|
| write|Write to a file or device.|
| close|Close the file or device.|
| ioctl|Pass control information to a device driver.|

The ioctl system call is used to provide some necessary hardware−specific control (as opposed to regular input and output), so its use varies from device to device.

### umask

- file default:666
- direction default:777

