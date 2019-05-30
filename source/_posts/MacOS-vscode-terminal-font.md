---
title: MacOS系统中vscode终端字体配置(终端显示异常)
date: 2019-05-10 09:56:29
categories:
- MacOS
- vscode
tags:
- MacOS
- vscode
---

# **Mac下配置vscode终端字体**

### 下载字体

```
cd /Library/Fonts
sudo git clone https://github.com/abertsch/Menlo-for-Powerline.git
```

### vscode中设置字体

- 使用快捷键`command+shift+p`,  输入`Open Settings(JSON)`

![image-20190510100611190](https://ws3.sinaimg.cn/large/006tNc79gy1g2vzun4fqqj30y407etad.jpg)

- 在JSON文件中添加以下字段

```
"editor.fontFamily": "Menlo for Powerline"
```

![image-20190510100803027](https://ws4.sinaimg.cn/large/006tNc79gy1g2vzwlfx8sj30mg060gm9.jpg)

### 效果

![image-20190510100136153](https://ws4.sinaimg.cn/large/006tNc79ly1g2vzpym1irj30pq05gdgs.jpg)

### 参考

[配置vscode终端字体](https://blog.csdn.net/chenghai37/article/details/81417293)

https://vscode.readthedocs.io/en/latest/getstarted/settings/

