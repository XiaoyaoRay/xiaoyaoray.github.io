---
title: Github上fork了别人的项目后，再同步更新别人的提交
copyright: true
date: 2018-04-30 09:44:47
tags:
- Github
---

## [github上fork了别人的项目后，再同步更新别人的提交](https://blog.csdn.net/qq1332479771/article/details/56087333)

我从github网站和用git命令两种方式说一下。

### **github网站上操作**

1. 打开自己的仓库，进入code下面。
2. 点击new pull request创建。 
   ![这里写图片描述](https://img-blog.csdn.net/20170221001500650?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXExMzMyNDc5Nzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
3. 选择base fork
4. 选择head fork
5. 点击Create pull request，并填写创建信息。

![这里写图片描述](https://img-blog.csdn.net/20170221001519228?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXExMzMyNDc5Nzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
![这里写图片描述](https://img-blog.csdn.net/20170226194654555?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXExMzMyNDc5Nzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
\6. 点击Merge pull request 合并从源fork来的代码。 
![这里写图片描述](https://img-blog.csdn.net/20170221001548713?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXExMzMyNDc5Nzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 
\7. 完成。

### **用git命令操作**

1. 用`git remote`查看远程主机状态

```
git remote -v 
git remote add upstream git@github.com:xxx/xxx.git
git fetch upstream
git merge upstream/master
git push 12345
```

![这里写图片描述](https://img-blog.csdn.net/20170221095054215?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXExMzMyNDc5Nzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](https://img-blog.csdn.net/20170221095105168?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXExMzMyNDc5Nzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](https://img-blog.csdn.net/20170221095114372?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXExMzMyNDc5Nzcx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)