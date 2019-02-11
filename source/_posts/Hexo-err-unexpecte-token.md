---
title:  Hexo 异常 - Template render error unexpected token
date: 2019-01-24 16:47:38
categories:
- Hexo
tags:
- Hexo
---

## 转载：[Hexo 异常 - Template render error unexpected token](https://hoxis.github.io/hexo-unexpected-token.html)

在使用 Hexo 调试时，一直出现 `{%raw%}Template render error: unexpected token: }}{%endraw%}` 的异常，显然是出现了特殊字符导致无法解析。

<!--more-->

### 现象

在上一篇 [Python 利用 Jinja2 模版生成文件](https://hoxis.github.io/python-jinja2.html) 中，每次执行 `hexo s` 或 `hexo g`，都会报错：

```
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Template render error: unexpected token: }}
    at new exports.TemplateError (E:\liuhao_data\hexo-blog\node_modules\hexo\node_modules\nunjucks\src\lib.js:51:19)
    at new_cls.fail (E:\liuhao_data\hexo-blog\node_modules\hexo\node_modules\nunjucks\src\parser.js:64:15)
    at new_cls.parsePrimary (E:\liuhao_data\hexo-blog\node_modules\hexo\node_modules\nunjucks\src\parser.js:947:18)
    at new_cls.parseUnary (E:\liuhao_data\hexo-blog\node_modules\hexo\node_modules\nunjucks\src\parser.js:882:25)
.......
```

### 原因

原文中有这句话：

> 先创建一个包括 `{%raw%}{{ }}{% endraw %}`或 `{% %}` 等特殊符号的模板文件

出错时 Markdown 原文是这样的：

```
先创建一个包括 `{{ }}`或 `{% %}` 等特殊符号的模板文件
```

其中 `{%raw%} {{ }} {%endraw%} `和 `{%raw%} {% %} {%endraw%} `被当成 hexo 模板中的标签，解析出错。

### 解决方法

Github 上给出的方法是在需要显示 `{%raw%}{{ }}{%endraw%}` 符号的地方用如下代码包围：

```
{% raw %}

{% endraw %}
```

标记这部分不需要解析。

- 修改后的 Markdown 原文

```
先创建一个包括 `{%raw%} {{ }} {%endraw%}` 或 `{%raw%} {% %} {%endraw%}` 等特殊符号的模板文件
```

- 解决后的效果

先创建一个包括 `{%raw%}{{ }}{%endraw%} `或 `{%raw%} {% %} {%endraw%} `等特殊符号的模板文件

虽然有点麻烦，但也算临时解决了这个问题，这是个已知 bug ，希望后续的版本能修复吧，毕竟使用太多 hexo 专属的标签对博客以后的迁移、改版什么的来说还是很麻烦的。

- 补充

用 ``` 包围的代码块不需要这样特殊处理。