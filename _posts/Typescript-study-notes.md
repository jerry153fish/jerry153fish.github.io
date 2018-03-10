---
layout: post
title: Typescript Study notes
key: 20180307
tags: Typescript Javascript Note
---

This is a collection of advanced javascript knowledge.

### Array

- map parameters


```js

['1', '2', '3'].map(parseInt) // [1, NaN, NaN]

['1', '2', '3'].map((currentValue, index, array) => {
  console.log(currentValue)
  console.log(index)
  console.log(array)
})
// parseInt(1, 0) == parseInt(1, 10) 1
// parseInt(2, 1) NaN
// parseInt(3, 2) NaN

```
- map will only called by initialized array element

```js
var ary = Array(3);
ary[0]=2
ary.map(function(elem) { return '1'; });

// ["1", empty × 2]
```


- filter will not work on empty elements


```js
var ary = [0,1,2];
ary[10] = 10;
ary.filter(function(x) { return x === undefined;});
```

- array < > will compare element ony by one but == and === will not

```js
var a = [1, 2, 3],
    b = [1, 2, 3],
    c = [1, 2, 4]
a ==  b // false
a === b // false
a > c // false
a < c // true
```
- array allow last one be comma

### Operation precedence

Precedence |	Operator type	| Associativity	| Individual operators
---|---|---|---
20	|Grouping	|n/a	|( … )
19	|Member Access	|left-to-right	|… . …
| |Computed Member Access|	left-to-right|	… [ … ]
| | new (with argument list) |	n/a	| new … ( … )
| | Function Call	| left-to-right	| … ( … )
18	|new (without argument list)|	right-to-left	| new …
17	|Postfix Increment	|n/a	| … ++
| |Postfix  Decrement	| |… --
16	| Logical  NOT	| right-to-left	| ! …
| |Bitwise  NOT	|| ~ …
| |Unary Plus|	| + …
| |Unary  Negation|	| - …
| |Prefix  Increment	|| ++ …
|| Prefix  Decrement	|| -- …
|| typeof ||	typeof …
|| void	|| void …
|| delete|	| delete …
|| await|	| await …
15	|Exponentiation	|right-to-left	| … ** …
14	| Multiplication	| left-to-right	| … * …
| | Division|	|… / …
|| Remainder	||… % …
13	|Addition	|left-to-right	| … + …
|| Subtraction	|| … - …
12	|Bitwise Left Shift	|left-to-right |	… << …
| |Bitwise Right Shift	||… >> …
| |Bitwise  Unsigned Right Shift ||	… >>> …
11	| Less Than	| left-to-right	| … < …
| |Less Than Or Equal||	… <= …
| |Greater Than	| |… > …
| |Greater Than Or Equal	|| … >= …
| | in	|| … in …
| |instanceof ||	… instanceof …
10	|Equality	|left-to-right	|… == …
| |Inequality||	… != …
| |Strict Equality ||	… === …
| |Strict Inequality ||	… !== …
9	| Bitwise AND	| left-to-right |	… & …
8	| Bitwise XOR	| left-to-right |	… ^ …
7	| Bitwise OR	| left-to-right |	… \| …
6	| Logical AND	| left-to-right	| … && …
5	|Logical OR	| left-to-right	| … \|\| …
4	|Conditional	| right-to-left	| … ? … : …
3	| Assignment	| right-to-left |	… = …
| | || … += …
| | | |… -= …
| | | |… **= …
| | | |… *= …
| | | |… /= …
| | | | %= …
| | | |… <<= …
| | | |… >>= …
| | | |… >>>= …
| | | |… &= …
| | | |… ^= …
| | | |… \|= …
2| 	yield	|right-to-left	|yield …
||yield*	||yield* …
1	| Comma \ Sequence	|left-to-right	|… , … 

```js

var val = 'smtg';
console.log('Value is ' + (val === 'smtg') ? 'Something' : 'Nothing'); // Something

```

### String

- new String will return String object

```js
var a = new String('a')
var b = String('a')

a === 'a' // false
b === 'a' // true
``` 

### == vs ===

- can not compare regex -- every regex is unique

```js
var a = /123/, b = /123/;
a == b // false
a === b // false
```

- == will perform type convent

```js
false == 0 // true
false == '0' // true
undefined == null // true
```

### prototype vs __proto__ vs Object.getPrototypeOf

1. prototype is a property of a Function object. It is the prototype of objects constructed by that function.

2. __proto__ is internal property of an object, pointing to its prototype. Current standards provide an equivalent Object.getPrototypeOf(O) method, though de facto standard __proto__ is quicker

3. Instanceof relationships by comparing a function's prototype to an object's __proto__ chain, and you can break these relationships by changing prototype.


```js

function Test() {
}

var t1 = new Test()

// below all true
t1.__proto__ === Test.prototype
t1.__proto__.__proto__ === Object.prototype
t1 instanceof Test
t1 instanceof Object
```



#### typeof vs instanceof

```js

typeof null // "object"
null instanceof Object // false
Object.prototype.toString.call(null) // '[Object Null]'

typeof [] // "object"
[] instanceof Object // true
Object.prototype.toString.call([]) // '[Object, Array]'

```

### hasOwnProperty will never look for proto chain

### function

- function name is read only