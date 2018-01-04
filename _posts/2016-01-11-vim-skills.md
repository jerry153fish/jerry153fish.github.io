---
layout: post
title: Useful Vim Skills
key: 20160111
tags: markdown
---

Useful Vim Skill notes 

### Vim Model

* Command Model

* Insert Model

* Last line Model

### Command Line

* To Insert model

  * i: insert before cursor
  * a: insert after cursor
  * I: insert at the head of the line
  * A: insert at the end of the line
  * o: open a new line below the cursor 
  * O: open a new line above the cursor

* Move

  * h j k l cursor move eg: 3h move cursor 3 left
  * gg move to start of file
  * G move to end of file
  * 0 start of the line
  * $ end of the line
  * w next start of word
  * e next end of word
  * b previous end of word 

* Edit 

  * d delete
  * r replace
  * c change
  * x delete cursor
  * u undo
  * y copy
  * p paste
  * ZZ save and exit
  * % jmp to {[()]}
  * > < indent
  * vi[close] close: {} [] () "" '' content insdie close
  * va[close] close: {} [] () "" '' "content" and close

### Insert Model

* ESC or Ctrl + c: go back go command model

### Last Line Model

* : under command model to enter last line model

#### quit

* q quit
* w save
* q! trash all change and quit

#### replace

- :s/aa/bb/g replace all aa with bb in current line
- :%s/aa/bb/g replace all aa with bb in current file
- :10,20s/aa/bb/g replace all aa with bb from line 10 to 20
- :10,20s/^/#/ insert # at start from 10 to 20
- :%s= *$==    delete all spaces end of all lines