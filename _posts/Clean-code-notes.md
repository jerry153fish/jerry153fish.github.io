---
layout: post
title: Clean code notes 
key: 20180319
tags: Note Clean-code
---

### Good code

- Elegant
- efficient
- complete error handing
    - memory leak
    - race condition
- must be **focus**
- readability
    - easy to read
    - direct
    - easy to enhance
- ties to tests
- smaller is better
- no duplication

### Meaningful Names

- use intention-revealing names

```java
int a; // not good

int daysSinceCreation; // good
```

- avoid disinformation
- make meaningful distinctions
- use pronounceable names
- use searchable names
- avoid encodings
    - hungarian notation
    - member prefixes
    - interface and implementation ??
- avoid mental mapping
    - need to clare

- Class names should have noun or noun phrase
- Method names should have verb or verb phrase
- Say what you mean and mean what you say
- one word per concept
- Do not pun
- use solution domain names
- use problem domain names
- Meaningful context
- do not add gratuitous context

### Functions

- small
    - less than 5 ?? ideally maybe
- do one thing
    - function should do one thing. they should do it well and they should do it only.

- one level of abstraction of function --- more practices
- the stepdown rule
    - every function to be followed by those at the next level of abstraction
- use polymorphism to bury switch to lower level
- use descriptive names

#### Function arguments

zero > one > two > three (better to stop here)

- use arguments object
- use arguments list
- avoid flag arguments
    - separate the function

#### try to avoid side effect

#### Command query separation

- function should either do something or answer something but not both
- prefer exceptions to returning error code
- extract try/catch blocks

```java
public doSomething(int num)
{
    try {
        doNumber(num);
    }
    catch (Exception e) {
        logError(e);
    }
}

```

#### DRY


### Comments

> Code explains itself

- necessary comments
    - legal comments
    - information comments
    - explanation of intent
    - clarification
    - warning of consequence
    - todos
    - amplification
    - javadocs for api

### Formatting

- in terms of team standard

### Objects and data structures










