---
layout: post
title: Shell Study Note 
key: 20160408
tags: shell linux bash
---

Linux shell variables tutorial.

### Variable

don't usually declare variables in the shell before we use them. Instead, we create them when we first use them get at the contents of a variable by preceding its name with a $ character and outputting its contents with the echo command

we need to give variables a preceding $, except when an assignment is being made to the variable

a string must be delimited by inverted commas if it contains spaces. Also note that there must be no spaces on either side of the equals sign.

$$ script id
$0 shell script name $1 2nd parameter $2 3rd parameter ..

$* A list of all the parameters, in a single variable, separated by the first character in the environment variable IFS.

$@ A subtle variation on $* ,that doesn't use the IFS environment variable.

```
$ IFS=''
$ set foo bar bam
$ echo "$@"
foo bar bam
$ echo "$*"
foobarbam
$ unset IFS
$ echo "$*"
foo bar bam
```

### Condition -- test or []

use quotes around the variable to avoid null = "some string"

### string

1. string1 = string2 true if equal
2. string1 != string2 true if not equal

### arithmetic

-eq -ne -lt -lt -gt -ge !

### structure control

```
for var in vars; do
  something
done

while condition; do
  something
done

until condition
do
statements
done
```

> $(( )) construct was a ksh invention, since included in the X/Open specification. Older shells will use expr instead

```
foo=$(($foo+1))
```

### case

```
case variable in
pattern [ | pattern] ...) statements;;
pattern [ | pattern] ...) statements;;
...
esac
```

```
#! /bin/sh
echo "Is it morning? Please answer yes or no"
read timeofday
case "$timeofday" in
yes) echo "Good Morning";;
no ) echo "Good Afternoon";;
y ) echo "Good Morning";;
n ) echo "Good Afternoon";;
* ) echo "Sorry, answer not recognized";;
esac
exit 0
```

Notice that each pattern line is terminated with double semicolons (;;).

[nN*] **pattern match**

### Command list

1. AND previous success excute next
2. OR Try unitl one success

> Function use $@ $* $1 $2 to get parameters

### Commands

1. build-in
2. self-defined
3. colon commands : ${var:=value} $(($var=value))

### Command execute

$() capture the result of command execute

### Parameter Expansion

${}

|Parameter Expansion|Description|
|---|---|
|${param:−default}| If param is null, set it to the value of default.|
|${#param}|Gives the length of param.|
|${param%word}|From the end, removes the smallest part of param that matches word and returns the rest.|
|${param%%word}|From the end, removes the longest part of param that matches word and returns the rest.|
|${param#word}|From the beginning, removes the smallest part of param that matches word and returns the rest.|
|${param##word}|From the beginning, removes the longest part of param that matches word and returns the rest.|

### eval

evaluate arguments

### set

sets the parameter variables for the shell.

```
set 1 2 3

$1:1 $2:2 ...

```

### shift

moves all the parameter variables down by one, so $2 becomes $1, $3 becomes $2, and so on.

### trap

The trap command is used for specifying the actions to take on receipt of signals

|Signal|Description|
|---|---|
|HUP (1)|Hang up; usually sent when a terminal goes off line, or a user logs out.|
|INT (2)|Interrupt; usually sent by pressing Ctrl−C.|
|QUIT (3)|Quit; usually sent by pressing Ctrl−\.|
|ABRT (6)|Abort; usually sent on some serious execution error.|
|ALRM (14)|Alarm; usually used for handling time−outs.|
|TERM (15)|Terminate; usually sent by the system when it's shutting down.|

### unset
The unset command removes variables or functions from the environment.

### here document
starts with the leader <<, followed by a special sequence of characters that will be repeated at the end of the document

### {} [] () in shell
1. ${var}
2. $(cmd) for command
3. in a serial of commands，commands within \` will be execude firstly and the result will be used as input

 ```
 $ ls
 a b c
 $ echo $(ls)
 a b c
 $ echo `ls`
 a b c
 ```
- () execude command in new shell
- {} execude command in current shell
- () and {} both puts command and use ; to seperate
- () last command could not use ;
- {}} last command must use ;
- {} must be space btween first comand and {
- () not need space between command and ()
- () and {} redirect in them only affect inside，but redirect outside affect all

### replac

#### ${var:-string},${var:+string},${var:=string},${var:?string}

- ${var:-string} and ${var:=string} if var is null string to replace ${var:-string}，if var not null，use var to replace ${var:-string}
- ${var:=string} is similar to ${var:-string}，if var is null, not only ${var:=string} become string，but only assign string to var

```
$ echo $newvar

$ echo ${newvar:-a}
a
$ echo $newvar ### newvar is null，but ${newvar:-a} become a

$ newvar=b
$ echo ${newvar:-a}
b
-------------------
$ echo $newvar

$ echo ${newvar:=a}
a
$ echo $newvar 
a
$ echo ${newvar:=b}
a
$ echo $newvar
a
```

- ${var:+string} on the other hand

```
$ echo $newvar
a
$ echo ${newvar:+b}
b
$ echo $newvar
a
$ newvar=
$ echo ${newvar:+b}
```

- ${var:?string}：if var not null，then use var as ${var:?string}；if var null，put string to stand error and exit。As as result, we can use to test var set or not

```
$ newvar=
$ echo ${newvar:?没有设置newvar的值}
bash: newvar: 没有设置newvar的值
$ newvar=a
$ echo ${newvar:?没有设置newvar的值}
a
```

#### ${var%pattern},${var%%pattern},${var#pattern},${var##pattern}

- ${var%pattern} from the end remove the shorest match
- ${var%%pattern} from the end remove the longest match
- ${var#pattern} from the beginning remove the shorest match
- ${var##pattern} from the beginning remove the longest match
- ${var/pattern/place} if match the first pattern will be replaced
- ${var//pattern/place} if match all the pattern will be replaced

### $((exp)) 

POSIX Standard :$((exp))

- any c operations can run inside，even ?:。note：not support float.if boolean operation，then true as 1, false as 0.

### change the messages of login

1. /etc/issue bash login welcome
2. /etc/motd ssh login welcome

### login and non-login

1. /etc/profile, /etc/profile.d/\*.sh login read
2. ~/.bash_profile login read
  - 1. ~/.bash_profile # read by order
  - 2. ~/.bash_login
  - 3. ~/.profile

![bash-process](/assets/img/bash_process.png)

3. ~/.bashrc (non-login shell read)

### terminals tty1~6

```
stty -a
```
### shell commands

reidction output >

```
ls -al > list.txt
```
append >>

2> operator. This is often useful to discard error information, to prevent it appearing on the screen.

```
kill −HUP 1234 > killout.txt 2>killerr.txt
```

### >& operator to combine the two outputs.

```
$ kill −1 1234 > killouterr.txt 2>&1
```

will put both the output and error outputs into the same file. Notice the order of the operators. This reads as
'redirect standard output to the file killouterr.txt, then direct standard error to the same place as the standard
output'.

redict input <