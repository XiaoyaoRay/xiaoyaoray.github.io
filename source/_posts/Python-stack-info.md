---
title: Python 调用堆栈信息
date: 2018-11-22 17:03:38
categories:
- Python
tags:
- Python
---

### [获取所有的方法和属性](http://stepsilon.com/python/python-get-all-methods-and-attribute-object-has)

For an object, e.g. named "myObj" you can get a list of all attributes with

```
dir(myObj)
```

and a list of all methods with

```
[method for method in dir(myObj) if callable(getattr(myObj, method))]
```

