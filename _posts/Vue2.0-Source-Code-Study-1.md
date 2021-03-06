---
layout: post
title: Vue 2.0 Source Code Study - Part One
key: 20180227
tags: Vue Source-code MVVM 
---

I have been using Vue 1.x and Vue 2.x for more than one year. I enjoy building apps in Vue so I decided to study the source code.

### Preparation

1. clone the source code to local

```sh

git clone git@github.com:vuejs/vue.git && cd vue

```

2. Find the earliest commits

```sh

git log --reverse

```

![vue commits](/assets/img/vue/commit.png)

3. Go to certain commit eg: a879ec06ef9504db8df2a19aac0d07609fe36131

    - View online 

    ```
    https://github.com/vuejs/vue/commit/a879ec06ef9504db8df2a19aac0d07609fe36131
    ```

    - Checkout locally

    ```
    git checkout -b source-read a879ec06ef9504db8df2a19aac0d07609fe36131
    ```


### init commit

Let us checkout locally and list the file structure

```sh
# tree .
├── build
│   └── build.js
├── package.json
├── src
│   ├── compiler
│   │   ├── codegen.js
│   │   ├── html-parser.js
│   │   ├── index.js
│   │   └── text-parser.js
│   ├── config.js
│   ├── index.js
│   ├── index.umd.js
│   ├── instance
│   │   └── index.js
│   ├── observer
│   │   ├── array.js
│   │   ├── batcher.js
│   │   ├── dep.js
│   │   ├── index.js
│   │   └── watcher.js
│   ├── util
│   │   ├── component.js
│   │   ├── debug.js
│   │   ├── dom.js
│   │   ├── env.js
│   │   ├── index.js
│   │   ├── lang.js
│   │   └── options.js
│   └── vdom
│       ├── dom.js
│       ├── h.js
│       ├── index.js
│       ├── modules
│       │   ├── attrs.js
│       │   ├── class.js
│       │   ├── events.js
│       │   ├── props.js
│       │   └── style.js
│       ├── patch.js
│       └── vnode.js
└── webpack.config.js

```












