---
title: RDS 2016 – Preview HTML5 RDP Client for Remote Desktop Services 2016
date: 2019-05-28 11:14:20
categories:
- Windows Server
- 2016
tags:
- Windows Server
---

## From [HTML5 RDP Client](https://c-nergy.be/blog/?p=11974)	

Hello World, 

In [our previous post](http://c-nergy.be/blog/?p=11950),  we have discussed the RDS roles in Windows 2019 Server Preview edition  and we have found out that some of the roles (the most important one  actually) are not available anymore which make not possible (at the  moment), to implement a full RDS 2019 Architecture.  While looking and  browsing the web and the Microsoft Documentations about RDS (remote  Desktop services), we came across this link Set up the [Remote Desktop web client for your users](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client-admin)

Reading through the documentation page,  it seems that in RDS 2016, a new RDP client is available.  This new RDP  client is not based on any ActiveX components but it’s a pure html 5 RDP  client. This was a request or a wish submitted on[ this web](https://remotedesktop.uservoice.com/forums/266795-remote-desktop-services/suggestions/8135250-modern-rdweb-access-with-html-5-support)  site so time ago.  It seems that this request went through…. Actually,  it seems that the RemoteApp Azure html client has been integrated in the  RDS 2016 solution 

In this post, we will try to install it  and see how it behave….We didn’t invent anything, all the necessary  information can be found [here](https://docs.microsoft.com/en-us/windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client-admin).  However, some information or guidance are not up to date anymore… 

Let’s do this … 

<!--more-->

## RDWeb Client Requirements and Installation

### Overview

The **Remote Desktop web client**  (also called RDP HTML5 Client) basically allows you to access the  Remote Desktop infrastructure through a compatible web browser.  Some  people might say : wait a minute, in previous version, we are also  accessing RDS infrastructure through a web interface via the RD Web  Access role, right?. So, what’s different ?  This is correct. However,  the previous RDS version needed Internet explorer and an activeX to have  a seamless experience.  The Internet explorer is calling the RDP client  installed on your Windows machine. 

The RDWeb client **does not need any ActiveX or any RDP client installed on the machine**.   A compatible browser  is what you need.  This means that you can  connect to remote Desktop sessions through other browsers like  Edge,Firefox,Chrome, and from different platform windows or linux as you  do not need a dedicated rdp client…

### Our Setup 

We have performed a quick installation  RDS scenario on a single server where the RD GW, Connection Broker and  RD Web access are installed on a single computer    

​	![image-20190530154230567](http://ww3.sinaimg.cn/large/006tNc79gy1g3jdyr9uspj31ne0u0k3u.jpg)

Notice also that we have generated  self-signed certificates for the RDS server roles for testing purposes.   As long as these certificates are trusted within your organization, you  should be able to test the RD Web client solution 

​	![image-20190530154415214](http://ww1.sinaimg.cn/large/006tNc79gy1g3je0kra1wj312z0u0aj3.jpg)

### Requirements

To be able to test and install the RDWeb client, you will need to ensure that the following requirements are met 

- your RDS Architecture must have the RD Gateway,RD connection broker and RD Web Roles installed on a Windows Server 2016
- Ensure that the licensing mode **is set per-user** and not per-device
- Clients connecting to your RDS infrastructure needs to be Windows 7 SP or later or Windows 2008 R2 or later
- Public certificates should be configured on RD Gateway and RD Web  Server (Preferred). Self signed certificate can be used in a lab or  internal environment.

*Note:*  

*The documentation also mention to install the [Windows 10 KB4025334](https://support.microsoft.com/en-us/help/4025334/windows-10-update-kb4025334) update on  the RD Gateway.  If your system is patched on a regular base, when  trying to install this patch, you might get an error stating that this  update is not applicable for your system.  If this is the case, simply  do not install the patch and proceed with the instructions*

### Update the PowerShellGet module  

 

- The RDWeb Client will be installed on  the RDS Server that has the RD Web Access Server role installed.   To  obtain and install the RDWeb Client package, perform the following  actions :

- On the RDWeb Access server, open the powershell command line (as an administrator) and type the following command 

  ```
  Register-PsRepository -Default
  Install-Module -Name PowerShellGet -Force
  ```

- You might get prompted to confirm your selection. press yes and wait for the installation to be completed. 

  ​       ![image-20190530155930711](http://ww4.sinaimg.cn/large/006tNc79gy1g3jegfzevaj31ai078mzz.jpg)

- When done, close and open again the PowerShell console with elevated rights  

### Download and Install the RDWeb client package 

To perform the installation, in the PowerShell windows (run as administrator), execute the following commands 

```
Install-Module -Name RDWebClientManagement
```

Confirm your selection; accept the license and wait for completion.   

​	![image-20190530160236276](http://ww4.sinaimg.cn/large/006tNc79gy1g3jejojenvj313n0u0q8d.jpg)

​	![image-20190530160304206](http://ww4.sinaimg.cn/large/006tNc79gy1g3jek779x7j313n0u0q8d.jpg)



Then execute the following command still in powershell 

```
Install-RDWebClientPackage
```

​    ![image-20190530160403988](http://ww1.sinaimg.cn/large/006tNc79gy1g3jel6ezqfj31bw0cgwgj.jpg)



You will see a warning about licensing  mode. Ignore it if you have already configured your system accordingly…  This is just a reminder

Finally, you will need to import the certificates of the RD  Connection broker into the RDWeb Access Server by executing the  following command 

```
Import-RDWebClientBrokerCert  <%Location of the certificate file (.cer format)%>
```

​	![image-20190530160514910](http://ww2.sinaimg.cn/large/006tNc79gy1g3jemeuqdbj31cc098mzg.jpg)



> **Quick Note**
>
> *To obtain the certificate, perform the following actions on the RD Connection broker server* 
>
> - *From an elevated command prompt, type mmc.exe*
> - *In the **mmc** console, on the top menu, select **file**> **add/remove snapins*** 
> - *In the **Add/Remove Snapins** dialog box, select on the left side, **Certificates** option and click on the **add>** button* 
> - *In the **certificates Snap-in** select the **Computer account** option and Press Finish* 
> - *Press OK in the Add/Remove Snapins Windows*
> - *In the mmc, navigate to Certificates > Personal > Certificates* 
> - *Select the correct certificate, right click on it and select All Tasks > export* 
> - *Follow the Wizard and be sure to save the file with the .cer extension* 

Finally, it’s time to publish the RD Web Client and you do this by executing the following command 

```
Publish-RDWebClientPackage  -Type Production -Latest
```

​	![image-20190530160654502](http://ww2.sinaimg.cn/large/006tNc79gy1g3jeo51m82j31c209i766.jpg)



Your installation is completed and now it’s time to test it….

## Accessing the RDWeb client Page and test it

To use the normal web interface of the  RDWeb access server role (the one with the ActiveX component), you would  open your browser and type the following (if no url redirection has  been set)

> **https://FQDN of the RDWeb Server/RDweb**  

To use the RDWeb client page,  you will need to use the following url 

> **https://FQDN of the RDWeb Server/RDWeb/Pages/webclient**

At this stage, you will reach the familar RD Web access login page 

​	![image-20190530160737455](http://ww3.sinaimg.cn/large/006tNc79gy1g3jeow6j9xj31n80u0tsk.jpg)



After providing the credentials, you will see the new RD Web Client interface 

​	![image-20190530160802485](http://ww1.sinaimg.cn/large/006tNc79gy1g3jepbfmv1j31gu0u0q8q.jpg)



If you click on one of the published  applications, you will get a first popup showing up asking you about  printing and clipboard options.  Select your options and press next  

​	![image-20190530160836462](http://ww1.sinaimg.cn/large/006tNc79gy1g3jepx0fstj31hj0u0qci.jpg)



then you will be asked for user name and password 

​	![image-20190530160901001](http://ww1.sinaimg.cn/large/006tNc79gy1g3jeqcflcxj31gw0u0dn5.jpg)



So, there is not real SSO capabilities  here.  You have to login to the interface and then when you start an  application for the first time, you will be prompted again for a  password.  This will happens only when you start the first application.   If you try to open another application just after the first one already  running, you will not get prompted for the password 

At this stage, you should access the  remote desktop session within your browser and you should see which  applications are available to you 

​	![image-20190530160931552](http://ww1.sinaimg.cn/large/006tNc79gy1g3jeqv5wxbj311l0u0k4u.jpg)



If you RDS collection is configured to  allow only Remote Desktop session, you can also have full Desktop access  through the Web Browser…..

​	![image-20190530161001296](http://ww4.sinaimg.cn/large/006tNc79gy1g3jere01bgj31i20soe7q.jpg)



For fun, we have tried to access the RD  Web Access server using the new RDWeb client from a Linux machine using  Firefox browser and no RDP client installed on it.  The results are  actually quite good and Linux users can now have access to Microsoft  applications through the Remote Desktop solution… Look pretty good …. 

​	![image-20190530161031152](http://ww1.sinaimg.cn/large/006tNc79gy1g3jerx3gbjj31ec0u0b2a.jpg)



 

## Final Notes

Keep in mind that this RDWeb client is a **preview version** and not final.  That means that maybe new features would be made available. 

​	![image-20190530161055790](http://ww1.sinaimg.cn/large/006tNc79gy1g3jesbzs8oj31gu0u0qby.jpg)



So far we have performed a really basic  test about this new option in the RDS 2016 Architecture solution. We  have to perform more testing in order to see if this could be a valid  option to be deployed organization wide or should we be keeping a the  standard approach with the ActiveX component.  We think that in the near  feature it will be a mix of both solutions.  Web client would be great  for Linux users and people using browsers other than Internet explorer. 

The RDWeb client is missing some  features or we might need to get used to a new way of working.  For  example, the web client does not provide (yet?) the drive redirection.  This might be by design in order to avoid data leakage.   Printing  feature is also changed apparently.  There is no need to redirect to a  printer.  The Web Client will generate a pdf file that can be printed  directly from the client machine…(data leakage issue ?). The RemoteApp  concept is a little bit different now.  Yes, with RDWeb client, it’s  possible to publish applications through a browser but the seamless  feature is gone.  Some organizations are really using this seamless  feature as a way to make application look like installed locally. 

We think that RDWeb client is a step in  the good direction because there is no more dependencies on Internet  Explorer and ActiveX components.  On the other hand, using only the RD  web client will definitly redefine the way we used to work with RDS  solution 

Till next time 

See ya





## ISSUES

### UnableToDownload https://go.microsoft.com/fwlink/?LinkID=627338&clcid=0x409



[To test this](https://answers.microsoft.com/en-us/windows/forum/windows_7-performance/trying-to-install-program-using-powershell-and/4c3ac2b2-ebd4-4b2a-a673-e283827da143) :

1. Open Powershell (As Admin)

2. [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

3. Try it again!

### Install PowerShellGet

https://docs.microsoft.com/zh-tw/powershell/scripting/gallery/installing-psget?view=powershell-7

安装时可以不使用`-Force`参数

PowerShell安装包查询：https://www.powershellgallery.com/packages



### Source Location 'https://www.powershellgallery.com/api/v2/package/RDWebclientManagement/1.0.3' is not valid.

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

