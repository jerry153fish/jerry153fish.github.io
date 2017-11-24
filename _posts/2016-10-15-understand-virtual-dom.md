---
layout: post
title: Understand Virtual Dom
key:   20161015
categories: websit
tags: virtual dom mvvm
comment: true
---

### Introduction

Why we need virtual dom? This is because dom is too heavy as it must implement at least those standards:

- [HTMLElement - Web API Interfaces](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)
- [Element - Web API Interfaces](https://developer.mozilla.org/en-US/docs/Web/API/Element)
- [GlobalEventHandlers](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers)

```
let p = document.createElement('p')
let keys = []
for (key in p) {
  keys.push(key)
}
console.log(keys.join(' '))
```
![heavyDom](/assets/img/domheavy.png)

On ther other hand, object in javascript is simple and light. Some geninus came out with Virtual DOM an abstraction of the HTML DOM.
eg:

```
let element = {
  tagName: 'div',
  props: { 
    class: 'father',
    id: 'top'
  },
  children: [
    {tagName: 'div', props: {class: 'son'}, children: ['1']},
    {
      tagName: 'div', props: {class: 'son'}, children: [
        {
          tagName: 'div', props: {class: 'grandson'}, children: ['2']}
        }
      ]
    },
  ]
}
// is the abstraction of 
<div class="father" id="top">
  <div class="son">1</div> 
  <div class="son">
    <div class="grandson">3</div>
  </div> 
</div>
```
Then we come to step one:

### Implementation

> Step one define create element method

```javascript
function createElement (tagName, props, children) {
  this.tagName = tagName
  this.props = props
  this.children = children
}
```
Then the example above could be represented as

```javascript
let el = createElement('div', { class: 'father'}, [
    createElement('div', { class: 'son'}, [
        '1'
    ])
    createElement('div', { class: 'grandson'}, [
        '3'
    ])
])
```

> render virtual dom into real

```javascript
createElement.prototype.render = function () {
  // create dom by tagmane
  var el = document.createElement(this.tagName)
  var props = this.props

  for (var propName in props) {
    // set dom attribute
    var propValue = props[propName]
    el.setAttribute(propName, propValue)
  }

  var children = this.children || []

  children.forEach(function (child) {
    var childEl = (child instanceof createElement) // check child is instance of createElement
      ? child.render()
      : document.createTextNode(child)
    el.appendChild(childEl)
  })

  return el
}
```

