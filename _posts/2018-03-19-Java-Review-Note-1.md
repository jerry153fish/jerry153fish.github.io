---
layout: post
title: Java Review Note 1
key: 20180306
tags: Java Note
---

### primitive type

- number
    - int : 4 byte
    - short : 2 byte
    - long : 8 byte
    - byte: 1 byte -127 ~ 127
    - no unsigned type in java

- float
    - float 4 byte
    - double 8 byte
    - float operation follow IEEE 754
        - Double.POSITIVE_INFINITY
        - Double.NEGATIVE_INFINITY
        - Double.NaN
- char: unicode
- boolean

- final vs const (c/c++/c#)

- type converting

![type converting](/assets/img/java/casting.png)

- string
    - immutable
    - String vs StringBuilder vs StringBuffer
    - in - `new Scanner(System.in)` vs `Console` which can only read one line
        - password visible vs array
- BigInteger and BigDecimal
    - valueOf
        - `BigInteger test = BigInteger.valueOf(100);`
    - can not use operation such as * /
        - Java not support operation override
        - need to use add or multiply

### Package and import

- package vs path