---
title: Git出现换行符报错(CRLT和LF)
copyright: true
date: 2018-06-01 10:01:52
tags:
- Git
---

### 问题出现原因

不同的操作系统(Windows和Linux)，所以换行符的差异会让 Git 认为文件的内容发生了变化。Windows下换行符为`CRLF`, Linux／Unix／Mac 系统换行符为`LF`

### 方法一

修改Git配置

```
git config core.filemode false
git config core.eol lf
```

### 方法二

1. 在 Windows 系统下，安装 Git 时应当置选项 `core.autocrlf` 为 `true`，也就是下图所示的那一个：
   ![请输入图片描述](http://segmentfault.com/img/bVck90)
2. 在 Linux／Unix／Mac 系统下，一般来说保持默认设置就好了（没有安装界面供选择）。当然你可以手工编辑 `~/.gitconfig` 设 `core.autocrlf` 为 `input`，如我机器上的设置：
   ![请输入图片描述](http://segmentfault.com/img/bVck98)

以上两种设置最终的结果就是提交的时候 Git 总是将换行符自动转为 `LF`，并且在 Windows 下 `checkout` 回来时自动帮你转换为 `CRLF`，以便适应 Windows 下的编辑器。

如果你做了上述设置依然会碰到题目中描述的问题，那么不用想，一定是你的团队中另外有别的成员没有做对以上两点的设置（很大的可能性是使用 Windows 的），你们可以逐一排查。

另外，安装 Windows Git 时另外一个设置也比较让人费解，建议选下图这样的：

![请输入图片描述](http://segmentfault.com/img/bVck99)

选择这种，所需要的配置量最少，CLI 和 Linux 的兼容性也比较高，唯一的缺陷就是 Windows CLI 里和 Linux 重名的命令会被 Linux 版本覆盖掉，比如 `find` 之类的。在我看来，反正 Windows CLI 是个渣渣，覆盖了也无所谓了。

### 参考

https://segmentfault.com/q/1010000000518769