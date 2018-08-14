---
title: 解决Windows10运行VMware Workstation出现与Device Guard不兼容导致无法运行与创建虚拟机问题
copyright: true
date: 2018-08-11 16:00:41
tags:
- VMWare
---

### 转载：[解决Windows10运行VMware Workstation出现与Device Guard不兼容导致无法运行与创建虚拟机问题](https://blog.minirplus.com/10268/)

最近在打开VMware Workstation虚拟机的时候突然发现无法新建和开启已有虚拟机，开始以为是在BIOS里关闭了Intel VT-x，但是检查一遍后，发现确实都已经开启了。研究了很久，以为是Device Guard的问题，但是最后发现，只是Hyper-V的问题，只需要关闭Hyper-V即可解决该问题。

<!--more-->

### 现象

运行已创建的虚拟机出现

> VMware Workstation 与 Device/Credential Guard 不兼容。在禁用 Device/Credential Guard 后，可以运行 VMware Workstation。有关更多详细信息，请访问http://www.vmware.com/go/turnoff_CG_DG。

[![img](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-35-04.jpg)](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-35-04.jpg)

新建虚拟机出现

> 此主机不支持64位客户机操作系统，此系统无法运行。

[![img](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-35-35.jpg)](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-35-35.jpg)

### 原因分析

Windows10开启Hyper-V后与VMware Workstation冲突导致无法运行和新建虚拟机。

一般来说Windows10默认不会打开Hyper-V，但是安装Docker默认会打开Hyper-V。

### 解决方法

禁用Hyper-V

### 步骤

打开Windows PowerShell（管理员）

[![img](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-47-52.jpg)](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-47-52.jpg)

运行命令

```
bcdedit /set hypervisorlaunchtype off
```

[![img](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-55-30.jpg)](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-55-30.jpg)

重启主机

[![img](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-56-07.jpg)](https://blog.minirplus.com/wp-content/uploads/2017/04/2017-04-19_21-56-07.jpg)

All Done！

### 总结

**如果实在不行，可以卸载Docker**

不要试图在控制面板>卸载程序>打开或关闭 Windows 功能中关闭 Hyper-V，否则在重启后会导致更新配置失败并回滚重启。

官方解决方案文档（[中文](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2148465)、[英文](https://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=2146361)）中的前面1-6步都是基于Itanium硬件平台，可以跳过不看，直接执行下面的基于Legacy BIOS boot的步骤，一共3步，但是中文的翻译有个坑，就是执行命令bcdedit /set hypervisorlaunchtypeoff 缺了一个空格，导致执行命令会报错，而英文版原文是正确的。

另外，由于VMware Workstation和Hyper-V冲突，那么就意味着VMware Workstation和Docker也冲突。

如果要重新开启Hyper-V，只需执行 bcdedit /set hypervisorlaunchtype auto 命令并重启即可。

虽然警告信息中显示与Device Guard不兼容，但是并不是，其实只是Hyper-V的问题