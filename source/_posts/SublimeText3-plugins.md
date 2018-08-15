---
typora-copy-images-to: ipic
title: Mac Sublime Text3 插件
copyright: true
categories:
- MacOS
tags: 
- Sublime Text3
- MacOS
---




### 安装插件步骤

- <span id="jump01">Sublime Text3</span> 中使用快捷键`command+shift+p`，在弹框中输入`install`后选择`Package Control: Intall Package`
<!--more-->
  ![image01](https://ws2.sinaimg.cn/large/006tNc79gy1fp33t7j8htj30d904zaae.jpg)

- 输入安装的插件名称，比如：`SublimeTmpl`

  ![image002](https://ws4.sinaimg.cn/large/006tNc79gy1fp33vxcqk3j30d40biwfd.jpg)

### SublimeTmpl

新建文件模板插件，可以支持多种语言例如Python、PHP等

- 安装参考[安装插件步骤](#jump01)

- 配置模板`python`为例，`Sublime Text-->Preferences-->Browse Packages`找到`SublimeTmpl`下`templates`中`python.tmpl`文件进行编辑

  ![Screen Shot 2018-03-06 at 15.25.56](https://ws4.sinaimg.cn/large/006tNc79gy1fp34paajchj30cd0a0mzf.jpg)

- 修改新建模板热键，在`Sublime Text-->Preferences-->Package Settings-->SublimeTmpl-->Key Bingdings -User`添加热键配置

  ```json
  {
          "keys": ["control+option+p"], "command": "sublime_tmpl", // windows key: "ctrl+alt+p"
          "args": {"type": "python"}, "context": [{"key": "sublime_tmpl.python"}]
      }
  ```

  ![Screen Shot 2018-03-06 at 15.45.29](https://ws2.sinaimg.cn/large/006tNc79gy1fp357fkqnmj30c909ljtf.jpg)

### Anaconda

自动匹配关键字等实用功能，有效提高开发效率

- 安装参考[安装插件步骤](#jump01)

### 附录

#### 问题1

- Build时报错**UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-2: ordinal not in range(128)**[网上链接](https://stackoverflow.com/questions/15166076/sublime-text-2-encoding-error-with-python3-build/15174760#15174760)

  ```json
  {
      "cmd": ["python3", "-u", "$file"],
      "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
      "selector": "source.python",
      "env": {"LANG": "en_US.UTF-8"}
  }
  ```


### 参考

[将Sublime Text 3设置为Python全栈开发环境](http://lib.csdn.net/article/python/63800)

[Sublime text 3 + python配置，完整搭建及常用插件安装](http://www.cnblogs.com/honkly/p/6599642.html)
