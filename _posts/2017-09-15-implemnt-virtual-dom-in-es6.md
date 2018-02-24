---
layout: post
title: Implement Virtual Dom In ES6
key:   20170915
tags: virtual DOM ES6 Journal
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

Now we know all the possible mutations of a new node, then we need to store those mutations for certain node. Thus we need to give each node a uid as well as a map for storage. This is because we can locate the mutations by

```js
let mutations = patch[uid]
```

eg: we change class name of a node. Then we generate with uid as 7. Then the mutations should be recorder as 

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
  let uid   = 0 // uid for identify the node
  let patches = {} 
  dfsTravel(oldTree, newTree, uid, patches)
  return patches
}
```

### Tree diff 

Now we know the node different, then we can move to the tree diff which is to compare nodes with same depth. Usually, a uid will be given to each node with Depth First Search. Moreover, when traveling with dfs, each node will also been compared accordingly. 

![dfswalks](/assets/img/toyvnodedfs.png)

There are different ways to give a uid to a node such as global uid implementation, hash method. The way in toy-v-node is very simple. The child uid = (total count of descendants of left sibing) + father uid + 1. Thus we need to count descendants of vdom before diff.

```js
    // inside ToyVDom class
    // this should be called only once when the root virtual dom is initialized
    countDescendants() {
        this.children.forEach(child => {
            if (child instanceof ToyVDom) {
                this.count += child.countDescendants()
            } 
            this.count++
        })
        return this.count
    }

    el.countDescendants()

```

![dfswalks](/assets/img/vdomCount.png)


The diff method will be


```js
function diff (oldTree, newTree) {
    let uid   = 0 // uid for identify the node
    let patches = {} 
    dfsTravel(oldTree, newTree, uid, patches)
    return patches
}

/**
 * dfsTravel Travel virtual node 
 * @param  { Object } oldVDom
 * @param  { Object } newVDom
 * @param  { Number } uid   - current old virtual dom unique id dfs position
 * @param  { Object } patches - map of patches
 */

function dfsTravel (oldVDom, newVDom, uid, patches) {
  let currentPatch = []

  // old virtual dom is removed
  if (newVDom === null || newVDom === undefined) {
    // no actions needed 
  } else if (isString(oldVDom) && isString(newVDom)) {
    // text virtual dom
    if (oldVDom !== newVDom) currentPatch.push({ type: TEXT, content: newVDom })
  } else if (
    // props diff
    oldVDom.tagName === newVDom.tagName &&
    oldVDom.key     === newVDom.key // key from should from props which identify the dom which different from uid
  ) {
    let propsPatches = diffProps(oldVDom, newVDom)
    if (propsPatches) {
      currentPatch.push({ type: PROPS, props: propsPatches })
    }
    // compare the child array
    diffChildren(oldVDom.children, newVDom.children, uid, patches)
  }
  else {
    currentPatch.push({ type: REPLACE, dom: newVDom})
  }

  if (currentPatch.length > 0) {
    patches[uid] = currentPatch
  }
} 

/**
 * diff with children array
 * @param  { Object } oldVDom
 * @param  { Object } newVDom
 * @param  { Number } uid   - father virtual dom unique id dfs position
 * @param  { Object } patches - map of patches
 * @param  { Array } currentPatch - array of patches contains mutations
 */
function diffChildren (oldChildren, newChildren, puid, patches, currentPatch) {
  let leftNode = null
  let currentNodeIndex = puid
  oldChildren.forEach(function (child, i) {
    let newChild = newChildren[i]
    currentNodeIndex = reproducdUUID(leftNode, puid) 
    dfsTravel(child, newChild, currentNodeIndex, patches) // dfs children array node
    leftNode = child
  })
}

/**
 * reproduce uid based on left node and parent uid
 * @param  { Object } leftNode
 * @param  { Number } uid   - father virtual dom unique id dfs position
 */
function reproducdUUID (leftNode, puid) {
  return (leftNode && leftNode.count) ? puid + leftNode.count + 1 : puid + 1 
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

function isString (value) {
  if (typeof value === 'string' || value instanceof String) {
    return true
  }
  return false
}
```


The code above can only perform replace, props change and text change.

![simple diff](/assets/img/simplediff.png)

We have two problems here:

* If the position of children was switched, for example, div, p switch to p, div. Then the diff method above will replace all of them. 

* Insert or remove mutations have not been implemented yet

Thus, we need to need a reorder the new virtual dom, which would move the order of new vdom according to old one and add the reorder mutations to the parent patches. 

```js
patches[puid] = [{
  type: REORDER,
  moves: [{remove or insert}, {remove or insert}, ...]
}]
```

### Reorder 

The children array might contains vdom with same tag name, thus it is not safe to identify vdom by tag names. Thus, we need a key (in props) to distinct same tag name vdoms.

There are three phase in the reorder methods:

1. flat the both children array to hash maps and array with no key vdoms

```js
function flatChildrenArray(children) {
    let keyHashMap = {}
    let noKeyItems = []
    for (let i = 0, len = children.length; i < len; i++) {
        let item = children[i]
        let itemKey = getKey(item)
        if (itemKey) {
            // key -- array index hash map
            keyHashMap[itemKey] = i
        } else {
            noKeyItems.push(item)
        }
    }
    return {
        keyHashMap: keyHashMap,
        noKeyItems: noKeyItems
    }
}

function getKey (vdom) {
    if (!vdom) return void 0
    let props = vdom.props
    if (props) {
        return props.key
    } else {
        return void 0
    }
}
```

The keyHashMap enable to trace back to the index of the children array. noKeyDoms keep vdom without keys.

2. create a reordered new children array from newChildren based on keys in oldChildren

```js
function generateReOrderNewChildren (oldChildren, newChildren, flatNewChildren) {
    let i = 0
    let reorderedNewChildren = []
    let newKeyHashMap = flatNewChildren.keyHashMap
    let noKeyItems = flatNewChildren.noKeyItems
    let noKeyIndex = 0
    while (i < oldChildren.length) {
        item = oldChildren[i]
        itemKey = getKey(item)
        if (itemKey) {
          if (!newKeyHashMap.hasOwnProperty(itemKey)) {
            reorderedNewChildren.push(null)
          } else {
            let newItemIndex = newKeyHashMap[itemKey]
            reorderedNewChildren.push(newChildren[newItemIndex])
          }
        } else {
          let noKeyItem = noKeyItems[noKeyIndex++]
          reorderedNewChildren.push(noKeyItem || null)
        }
        i++
    }
    return reorderedNewChildren
}
```

Now the reorderedNewChildren have same order and length with old children

3. format the reorderedNewChildren: remove null dom and add to moves mutations

```js
function clearRecorderNewChildrenAndRecordRemove (reorderedNewChildren, moves) {
    let i = 0
    let formatedRecordNChildren = [...reorderedNewChildren]
    while (i < formatedRecordNChildren.length) {
      if (formatedRecordNChildren[i] === null) {
        recordRemove(i, moves)
        removeArrayByIndex(formatedRecordNChildren, i)
      } else {
        i++
      }
    }
    return formatedRecordNChildren
}
```

4. compare formatedNewChildrenArray with newChildren array, records insert, moves mutations

```js
function generateInsertMoveMutations (formatedRecordNChildren, newChildren, flatOldChildren, moves) {
  let j = i = 0
  let oldKeyHashMap = flatOldChildren.keyHashMap
  while (i < newChildren.length) {
    item = newChildren[i]
    itemKey = getKey(item)

    let formatedRecorderedItem = formatedRecordNChildren[j]
    let formatedRecorderedItemKey = getKey(formatedRecorderedItem)

    if (formatedRecorderedItem) {
      if (itemKey === formatedRecorderedItemKey) {
        j++
      } else {
        // new item, recorder insert
        if (!oldKeyHashMap.hasOwnProperty(itemKey)) {
          recordInsert(i, item, moves)
        } else {
          // remove formatedrecorder array if next one is equal to new children
          let nextformatedRecorderedItemKey = getKey(formatedRecordNChildren[j + 1])
          if (nextformatedRecorderedItemKey === itemKey) {
            recordRemove(i, moves)
            removeArrayByIndex(formatedRecordNChildren, j)
            j++ // after removing, current j is right, just jump to next one
          } else {
            // else insert item
            recordInsert(i, item, moves)
          }
        }
      }
    } else {
      recordInsert(i, item, moves)
    }
    i++
  }
}
```

At this stage we records all reorder moves, the reorder method is completed. All the moves and the sorted tem children array for virtual dom diff will be returned.

```js

return {
  moves: moves,
  reorderedNewChildren: reorderedNewChildren
}
```

Then we can update our diffchildren method

```js
function diffChildren (oldChildren, newChildren, puid, patches, currentPatch) {
  let reorders = reorder(oldChildren, newChildren)
  newChildren = reorders.children

  if (reorders.moves.length) {
    let reorderPatch = { type: REORDER, moves: reorders.moves }
    currentPatch.push(reorderPatch)
  }
  let leftNode = null
  let currentNodeIndex = puid
  oldChildren.forEach(function (child, i) {
    let newChild = newChildren[i]
    currentNodeIndex = reproducdUUID(v, puid) 
    dfsTravel(child, newChild, currentNodeIndex, patches) // dfs children array node
    leftNode = child
  })
}
```
### Patch

Now we have record all the mutations in a hashmap: patches

![vdompatch](/assets/img/vdompatch.png)

The patch method updates patches to old read dom, it is very similar to diff. As the real dom tree have the same structure as the old virtual dom. 

```js
function patch (realDom, patches) {
  let walker = {uid: 0}
  dfsTravelRDom(realDom, walker, patches)
}

function dfsTravelRDom (realDom, walker, patches) {
  // get current patches
  let currentPatches = patches[walker.uid]

  let len = realDom.childNodes
    ? realDom.childNodes.length
    : 0
  // travel children array
  for (var i = 0; i < len; i++) {
    var child = realDom.childNodes[i]
    walker.uid++
    dfsTravelRDom(child, walker, patches)
  }
  // apply patch
  if (currentPatches) {
    updatePatches(realDom, currentPatches) // 对当前节点进行DOM操作
  }
}

// update patches
function updatePatches (realDom, currentPatches) {
  currentPatches.forEach(function (currentPatch) {
    switch (currentPatch.type) {
      case REPLACE:
        replaceDom(realDom, currentPatch.dom)
        break
      case REORDER:
        reorderChildren(realDom, currentPatch.moves)
        break
      case PROPS:
        setProps(realDom, currentPatch.props)
        break
      case TEXT:
        node.textContent = currentPatch.content
        break
    }
  })
}

```

check the detailed [patch.js](https://github.com/jerry153fish/toy-v-dom)

### Conclusion

The virtual dom would greatly improve the performance in MVVM framework. The can be achieved in three steps:

1. virtual dom render as real dom
2. compare old virtual dom with new virtual dom and find the mutations
3. update those differences to the real dom


### Reference

[^1]: Philip B 2001, **A Survey on Tree Edit Distance and Related Problems**,https://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf

[^2]: React 2018, **Reconciliation**, https://reactjs.org/docs/reconciliation.html
