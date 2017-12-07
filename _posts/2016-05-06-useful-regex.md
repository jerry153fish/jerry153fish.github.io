---
layout: post
title: Useful Regex
key: 20160506
tags: regex
---

### Useful regex

> check password must include number and uppercase and lowercase no special characters

```
^(?=.*\\d)(?=.*[a-z])(?=.*[A-Z]).{8,10}$
```

> check chinese

```
^[\\u4e00-\\u9fa5]{0,}$
```

> check email

```
[\\w!#$%&'*+/=?^_`{|}~-]+(?:\\.[\\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\\w](?:[\\w-]*[\\w])?\\.)+[\\w](?:[\\w-]*[\\w])?
```

> check url

```
^(f|ht){1}(tp|tps):\\/\\/([\\w-]+\\.)+[\\w-]+(\\/[\\w- ./?%&=]*)?
```
