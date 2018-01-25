---
layout: post
title: Implement Virtual Dom In ES6
key:   20170915
tags: virtual dom ES6
mathjax: true
---

In order to understand MVVM, I was trying to revert wheel for virtual dom which is essential part for MVVM framework such as reactjs and vuejs.

To begin with, we need to know why virtual dom? This is because dom is too heavy as it must implement at least those standards:

- [HTMLElement - Web API Interfaces](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)
- [Element - Web API Interfaces](https://developer.mozilla.org/en-US/docs/Web/API/Element)
- [GlobalEventHandlers](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers)

```js
let p = document.createElement('p')
let keys = []
for (key in p) {
  keys.push(key)
}
console.log(keys.join(' '))
```
![heavyDom](/assets/img/domheavy.png)

On the other hand, object in javascript is simple and light. Some people came out with Virtual DOM an abstraction of the HTML DOM.
eg:

```js
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
<div class="root" id="top">
  <p class="son">1</p> 
  <div class="son">
    <p class="grandson ">2</p>
  </div> 
</div>
```
Then we come to step one:

### Virtual Dom implementation 

> Step one define create element method

```js
class ToyVDom () {
  constructor(tagName, props, children) {
    this.tagName = tagName
    this.props = props
    this.children = children
  }
}
``` 
Then the example above could be represented as

```js
let el = new ToyVDom('div', { class: 'root', id: 'top'}, [
    new ToyVDom('p', { class: 'son'}, [
        '1'
    ]),
    new ToyVDom('div', { class: 'son'}, [
        new ToyVDom('p', { class: 'grandson '}, ['2'])
    ])
])
```

> render virtual dom into real DOM

```js

class ToyVDom {
  constructor(tagName, props, children) {
    this.tagName = tagName
    this.props = props
    this.children = children
  }
  render () {
    // create dom by tag mane
    let el = document.createElement(this.tagName)
    let props = this.props

    for (let propName in props) {
      // set dom attribute
      let propValue = props[propName]
      el.setAttribute(propName, propValue)
    }

    let children = this.children || []

    children.forEach(function (child) {
      let childEl = (child instanceof ToyVDom) // check child is instance of createElement
        ? child.render()
        : document.createTextNode(child)
      el.appendChild(childEl)
    })
    return el
  }
}

```

Test:

```js
let root = el.render()
document.body.append(root)
``` 
![virtual dom test](/assets/img/virtualDomTest.png)

### Diff

Now we know how to create the virtual dom. Then we need to manipulate the virtual dom. Thus, we need a diff algorithm compare two dom tree and find the difference actions such as updating, deleting can be implemented.

> According to he diff algorithm is a Tree Edit Distance and Related Problems[^1] whose time complexity is a O(n^3).

However, we can reduce time complexity O(n) based on the two assumption [^2]

1. Two elements of different types will produce different trees.
2. The developer can hint at which child elements may be stable across different renders with a key prop.

In another words, we index each nodes and compare them on the save depth.

![vdom layer to layer](http://res.cloudinary.com/oyecode/image/upload/v1442798562/React%20JS%20diff%20Algo.png)

### Node diff

When comparing nodes, there are four possible situations:

1. different type of element: tagName is different
2. different props of element: props is different
3. text is different
4. order of element is different: children arrays of different order

Then we can define constrain for mutations

```js
const REPLACE = 0  // replace => 0
const PROPS   = 1  // props   => 1
const TEXT    = 2  // text    => 2
const REORDER = 3  // reorder => 3
```

Now we know all the possible mutations of a new node, then we need to store those mutations for certain node. Thus we need to give each node a uuid as well as a map for storage. This is because we can locate the mutations by

```js
let mutations = patch[uuid]
```

eg: we change class name of a node. Then we generate with uuid as 7. Then the mutations should be recorder as 

```js
patch = {
  7 : [
    {
      type: ATTRS,
      props: { class: 'new class' }
    }
  ]
}
```

The diff function entry

```js

function diff (oldTree, newTree) {
  let uuid   = 0 // uuid for identify the node
  let patches = {} 
  dfsTravel(oldTree, newTree, uuid, patches)
  return patches
}
```

### Tree diff 

Now we know the node different, then we can move to the tree diff which is to compare nodes with same depth. Usually, a uuid will be given to each node with Depth First Search. Moreover, when traveling with dfs, each node will also been compared accordingly. 

![dfswalks](/assets/img/toyvnodedfs.png)


```js
import { is } from 'ramda' // import is type
/**
 * dfsTravel Travel virtual node 
 * @param  { Object } oldVDom
 * @param  { Object } newVDom
 * @param  { Number } uuid   - current old virtual dom unique id
 * @param  { Object } patches - map of patches
 */
function dfsTravel (oldVDom, newVDom, uuid, patches) {
  let currentPatch = []

  // old virtual dom is removed
  if (newVDom === null || newVDom === undefined) {
    // no actions needed 
  } else if (is(String, oldVDom) && is(String, newVDom)) {
    // text virtual dom
    if (oldVDom !== newVDom) currentPatch.push({ type: TEXT, content: newNode })
  } else if (
    // props diff
    oldVDom.tagName === newVDom.tagName &&
    oldVDom.key     === newVDom.key // key from should from props which identify the dom which different from uuid
  ) {
    let propsPatches = diffProps(oldVDom, newVDom)
    if (propsPatches) {
      currentPatch.push({ type: ATTRS, props: propsPatches })
    }
    // compare the child array
    diffChildren(oldVDom.children, newVDom.children, uuid, patches)
  }
  else {
    currentPatch.push({ type: REPLACE, node: newVDom})
  }

  if (currentPatch.length > 0) {
    patches[uuid] = currentPatch
  }
} 

// diff with children
function diffChildren (oldChildren, newChildren, uuid, patches, currentPatch) {
  let leftNode = null
  let currentNodeIndex = uuid
  oldChildren.forEach(function (child, i) {
    let newChild = newChildren[i]
    currentNodeIndex = dfsUuid(v, uuid) 
    dfsWalk(child, newChild, currentNodeIndex, patches) // dfs children array node
    leftNode = child
  })
}

// count descentdants of a virutal dom
function countDescendants (vdom, count) {
  if (vdom) {
    vdom.children.foreach(v => {
      if (v instanceof ToyVDom) {
        count += countDescendants(vdom, count) 
      }
      count ++
    })
  }
  return count
}

// get dfs uuid of a virtual dom from left node and parent uuid 
function dfsUuid (leftNode, puuid) {
  let leftNodeDescendants = countDescendants(leftNode, 0)
  return leftNodeDescendants ? puuid + leftNodeDescendants + 1 : puuid + 1 
}

// diff Attributes function
function diffProps (oldVDom, newVDom) {
  let count    = 0
  let oldProps = oldVDom.props
  let newProps = newVDom.props

  let key, value
  let propsPatches = {}

  // check every attribute 
  for (key in oldProps) {
    value = oldProps[key]
    // props been removed then = undefined
    if (newProps[key] !== value) {
      count++
      propsPatches[key] = newProps[key]
    }
  }
  // new props
  for (key in newProps) {
    value = newProps[key]
    if (!oldProps.hasOwnProperty(key)) {
      count++
      propsPatches[key] = value
    }
  }

  if (count === 0) {
    return null
  }

  return propsPatches
}
```

### list diff



```js

```

In this simple implementation, the global uuid only ensure 











### Reference

[^1]: Philip B 2001, **A Survey on Tree Edit Distance and Related Problems**,https://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf

[^2]: React 2018, **Reconciliation**, https://reactjs.org/docs/reconciliation.html
