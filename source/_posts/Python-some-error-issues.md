---
title: Python报错问题解决办法收集
date: 2018-10-10 18:09:44
categories:
- Python
tags:
- Python
---

## coercing to Unicode: need string or buffer, xxxxx

### 产生原因

字符串用`+`拼接时，如果非字符串类型（比如：None，tuple，list等）会产生该错误，比如以下代码

```
test_str1 = "test11" + None
test_str2 = "test22" + [1,2]
test_str3 = "test33" + (1,2)
```

### 解决办法

改用%操作符格式化输出，如以下代码

```
test_str1 = "test11 %s " % None
test_str2 = "test22 %s" % [1,2]
```

