---
layout: post
title:  Introduction to flexbox
tags: flex layout css3
key: 20160923
tags: flexbox flex css3
comment: true
---

### Why flex

Flex enable containers to adust items' width/height/order to best fill the space in different kinds of screen sizes in a simple way.

### How to use

```
.flex-element {
  display: -webkit-flex; /* Webkit core browerer such as Safari */
  display: flex;
}
```
Then all the children elements will lose **float**, **clear**,**vertical-align** properties.

The layout of a flex container looks like below:
![flex Introduction]({{site.baseurl}}/assets/img/flex-introduce.png)

There are six properties in a flex container:

> flex-direction -- How items are placed

  ```
    flex-direction: row | row-reverse | column | column-reverse;
  ```

<iframe height='265' scrolling='no' src='//codepen.io/jerry153fish/embed/JRBGQZ/?height=265&theme-id=0&default-tab=html&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/jerry153fish/pen/JRBGQZ/'>JRBGQZ</a> by jerry153fish (<a href='http://codepen.io/jerry153fish'>@jerry153fish</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

> flex-wrap How items are fitted

```
  flex-wrap: nowrap | wrap | wrap-reverse;
```

<iframe height='265' scrolling='no' src='//codepen.io/jerry153fish/embed/GjBqPB/?height=265&theme-id=0&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/jerry153fish/pen/GjBqPB/'>flex-wrap</a> by jerry153fish (<a href='http://codepen.io/jerry153fish'>@jerry153fish</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>


> flex-flow conbine flex-direction and flex-wrap


```
  flex-flow: <flex-direction> || <flex-wrap>;
```


4. justify-content How items align along the main axis


```
  justify-content: flex-start | flex-end | center | space-between | space-around;
```

<iframe height='265' scrolling='no' src='//codepen.io/jerry153fish/embed/LRJjmd/?height=265&theme-id=0&default-tab=html,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/jerry153fish/pen/LRJjmd/'>justify-content</a> by jerry153fish (<a href='http://codepen.io/jerry153fish'>@jerry153fish</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

