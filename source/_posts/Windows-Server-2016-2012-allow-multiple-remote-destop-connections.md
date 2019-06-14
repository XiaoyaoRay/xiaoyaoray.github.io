---
title: Allow Multiple Remote Desktop (RDP) Connections In Windows Server 2012/2016
date: 2019-06-03 15:11:45
categories:
- Windows Server
- 2016
tags:
- Windows Server
---

## From [Multiple Remote Desktop (RDP) Connections](https://www.hostwinds.com/guide/allow-multiple-remote-desktop-rdp-connections-in-windows-server-2012-2016/)

This guide will explain how to allow multiple sessions in a Windows VPS. A common usage case for this would be to allow the usage of a developed application or a remotely accessed support desktop. Additionally, there will be some information on where to purchase the RDS CAL license directly from Microsoft to allow for a larger number of users.

If you need more than 2 connections to your Windows Server 2016 from internal device, then, below RDS serve role might be necessary:
1. RD Connection Broker.
2. RD Session Host.
3. RD Licensing server – there is 120-day licensing grace period, appropriate RDS CALs should be purchased and installed before it has expired.

*It may be best to consider a higher specification VPS or a Dedicated Server if you wish to use more than 3 users per machine, as the overall performance of the VPS will degrade per user.*

<!--more-->

## How To Enable Multiple Remote Desktop (RDP) Sessions

**Step 1.** Connect to the Windows Server session by RDP.

**Step 2.** Click The Search button next to the start menu (Windows 2016) or typing into the start menu (Windows Server 2012)

 ![The Start Menu where you can type into to search.](http://ww1.sinaimg.cn/large/006tNc79ly1g3nzm6ti1aj30hq0iydj5.jpg)



**Step 3.** Enter in **gpedit.msc**

 ![The Search Menu with gpedit.msc/Edit Group Policy highlighted.](http://ww4.sinaimg.cn/large/006tNc79gy1g3nzvbm5zjj30ay0iyq3e.jpg)



**Step 4.** Once Group Policy Editor had loaded navigate to **Computer Configuration**, next **Administrative Templates**, then **Windows Components**, then **Remote Desktop Services**, then **Remote Desktop Session host**.

![The Group Policy Editor with the first half of the path marked.](http://ww1.sinaimg.cn/large/006tNc79gy1g3nzv0r940j30r80h23zq.jpg)

![The Group Policy Editor with the second half of the path edited.](http://ww1.sinaimg.cn/large/006tNc79gy1g3nzuubrp0j30r80h2jsl.jpg)



**Step 5.** From there, you should see a folder marked as ***Connections***, click into it.

**Step 6.** Next right click with your mouse on **Limit Number of Connections** and click ***Edit***.

![The particular policies after the changes. Before changes, it will say "Not Configured"](http://ww3.sinaimg.cn/large/006tNc79gy1g3nzu4nsttj30s30e3dgl.jpg)



**Step 7.** From there, you can set the number to the limit you wish to have, or turn it off.

 ![Here you can change it from disabled, enabled, or Not Configured. ](http://ww3.sinaimg.cn/large/006tNc79gy1g3nzt95kqvj30j40hqgm3.jpg)



*For access to more than two sessions at a time, you will want to purchase a RDS Cal license from a certified provider. You can purchase a Remote Desktop Service Client Access License from Microsoft.*



**Step 8.** Next, click Next Setting until you are at the **Restrict Remote Desktop Services users to a single Remote Desktop Services session** screen so you can Edit this setting.

 ![This window is simpler, and requests for you to enable, disable, or leave unconfigured. To allow more than once user, you can set it to enabled.](http://ww1.sinaimg.cn/large/006tNc79ly1g3nzmazghaj30j40hqmxp.jpg)



**Step 9.** In this window, you can click ***Disabled*** to turn off the user restrictions.

**Step 10.** Finally, [reboot the server](https://www.hostwinds.com/guide/server-management-panel-overview/) from your Cloud Control Overview page and the group policy changes should automatically apply.



## Reversing These Changes

The process to reverse the changes is easy. You would follow the same steps above, and set the desired Group Policies to Not Configured and select **Enabled** or **Disabled**.

## Reference

[Allow Multiple Remote Desktop (RDP) Connections In Windows Server 2012/2016](https://www.hostwinds.com/guide/allow-multiple-remote-desktop-rdp-connections-in-windows-server-2012-2016/)

[Easiest way to enable more than 2 concurrent RDP sessions on Windows Server 2016](https://social.technet.microsoft.com/Forums/en-US/c52b6da7-cdf2-4e1f-95d1-6471c6f2f6b0/easiest-way-to-enable-more-than-2-concurrent-rdp-sessions-on-windows-server-2016?forum=winserverTS)

[Terminal Server 2016 And How To Get More Out Of It | Parallels Explains](https://www.parallels.com/blogs/ras/terminal-server-2016/)

