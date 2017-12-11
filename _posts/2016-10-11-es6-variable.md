---
layout: post
title: ES6 Variable - ES6 Study
key:   20161011
categories: website
tags: es6 variable study note
comment: true
---

ES6 variable study notes.

### block level variable

```
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

### No variable lift up

```
console.log(foo); // 输出undefined
console.log(bar); // 报错ReferenceError

var foo = 2;
let bar = 2;
```

### variable blind in block temporal dead zone，aka TDZ

```
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

### no repeative declaire

```
// error
function () {
  let a = 10;
  var a = 1;
}

// error
function () {
  let a = 10;
  let a = 1;
}
```
### function declare

- According to ES6, function declared inside block similar to let

### let declared in global will not add to global
```
var a = 1;
// in node，global.a
// or this.a
window.a // 1

let b = 1;
window.b // undefined
```

### Destructuring
```
let [a, b, c] = [1, 2, 3]
// a = 1 b = 2 c =3

// right side must be an iterator
```

### Use shorthand in objet

```
const maxAge = 100
const person = {
    name: 'name',
    age: 3,
    // use this
    maxAge,
    // not this
    badAge: maxAge,
    // use this
    getName () {
        return this.name
    }
    // not use this
    getAge: function () {
        return this.age
    }
}
```

### Spread

```
let jerry = { name: 'Jerry', age: '30' }
// copy
let anotherJerry = {...jerry}
// change
let yongerJerry = {...jerry, age: 16}
// delete
let {name, ...age} = jerry
```