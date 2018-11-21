---
title: 配置 Securing OpenSSH
date: 2018-11-19 09:51:44
categories:
- Linux
tags:
- Linux
- CentOS
---



## 转载：[官网Securing OpenSSH](https://wiki.centos.org/HowTos/Network/SecuringSSH)



OpenSSH (or Secure SHell) has become a *de facto* standard for remote access replacing the telnet protocol. SSH has made protocols such as telnet redundant due, in most part, to the fact that the connection is encrypted and passwords are no longer sent in plain text for all to see.

However, a default installation of ssh isn't perfect, and when running an ssh server there are a few simple steps that can dramatically harden an installation.



## Use Strong Passwords/Usernames



One of the first things you'll notice if you have ssh running and exposed to the outside world is that you'll probably log attempts by hackers to guess your username/password. Typically a hacker will scan for port 22 (the default port on which ssh listens) to find machines with ssh running, and then attempt a brute-force attack against it. With strong passwords in place, hopefully any attack will be logged and noticed before it can succeed.

Hopefully you already use strong passwords, but if you are not then try to choose passwords that contains:

- Minimum of 8 characters
- Mix of upper and lower case letters
- Mix of letters and numbers
- Non alphanumeric characters (e.g. special characters such as ! " £ $ % ^ etc)

The benefits of strong passwords aren't specific to ssh, but have an impact on all aspects of systems security. Further information on passwords can be found in the CentOS documentation:

<http://www.centos.org/docs/4/html/rhel-sg-en-4/s1-wstation-pass.html>

If you absolutely can't prevent your users choosing weak passwords, then consider using randomly generated or difficult to guess usernames for your user accounts. If the bad guys can't guess the username then they can't brute force the password. However, this is still security through obscurity and be aware of information leakage of usernames from things such as email sent from user accounts.



## Disable Root Logins



SSH server settings are stored in the **/etc/ssh/sshd_config** file. To disable root logins, make sure you have the following entry:



```
# Prevent root logins:
PermitRootLogin no
```



and restart the sshd service:



```
service sshd restart 
```



If you need root access, login as a normal user and use the **su** command.



## Limit User Logins



SSH logins can be limited to only certain users who need remote access. If you have many user accounts on the system then it makes sense to limit remote access to only those that really need it thus limiting the impact of a casual user having a weak password. Add an AllowUsers line followed by a space separated list of usernames to **/etc/ssh/sshd_config** For example:



```
AllowUsers alice bob
```



and restart the sshd service.



## Disable Protocol 1



SSH has two protocols it may use, protocol 1 and protocol 2. The older protocol 1 is less secure and should be disabled unless you know that you specifically require it. Look for the following line in the **/etc/ssh/sshd_config** file, uncomment it and amend as shown:



```
# Protocol 2,1
Protocol 2 
```



and restart the sshd service.



## Use a Non-Standard Port



By default, ssh listens for incoming connections on port 22. For a hacker to determine ssh is running on your machine, he'll most likely scan port 22 to determine this. An effective method is to run ssh on a non-standard port. Any unused port will do, although one above 1024 is preferable. Many people choose 2222 as an alternative port (as it's easy to remember), just as 8080 is often known as the alternative HTTP port. For this very reason, it's probably not the best choice, as any hacker scanning port 22 will likely also be scanning port 2222 just for good measure. It's better to pick some random high port that's not used for any known services. To make the change, add a line like this to your **/etc/ssh/sshd_config** file:



```
# Run ssh on a non-standard port:
Port 2345  #Change me
```



and restart the sshd service. Don't forget to then make any necessary changes to port forwarding in your router and any applicable firewall rules. For example on CentOS 7 you need to change firewalld:



```
$ firewall-cmd --add-port 2345/tcp
$ firewall-cmd --add-port 2345/tcp --permanent
```



Or on CentOS 6:

```
$ iptables -I INPUT -p tcp --dport 2345 -j ACCEPT
```





On CentOS 6 and above you should also update selinux, labeling the chosen port correctly, otherwise sshd will be prevented from accessing it. For example:



```
$ semanage port -a -t ssh_port_t -p tcp 2345 #Change me 
```



Because ssh is no longer listening for connections on the standard port, you will need to tell your client what port to connect on. Using the ssh client from the command line, we may specify the port using the -p switch:



```
$ ssh -p 2345 myserver
```



or if you are using the fish protocol in konqueror, for example:



```
fish://myserver:2345/remote/dir
```



If you are thinking that this sounds like a pain having to specify the port each time you connect, simply add an entry specifying the port in your local **~/.ssh/config** file:



```
 # Client ~/.ssh/config
Host myserver
HostName 72.232.194.162
    User bob
    Port 2345 
```



And the file: **~/.ssh/config** must have the following permissions:



```
$ chmod 600 ~/.ssh/config 
```





## Filter SSH at the Firewall



If you only need remote access from one IP address (say from work to your home server), then consider filtering connections at your firewall by either adding a firewall rule on your router or in iptables to limit access on port 22 to only that specific IP address. For example, in iptables this could be achieved with the following type of rule:



```
iptables -A INPUT -p tcp -s 72.232.194.162 --dport 22 -j ACCEPT
```



SSH also natively supports TCP wrappers and access to the ssh service may be similarly controlled using hosts.allow and hosts.deny.

If you are unable to limit source IP addresses, and must open the ssh port globally, then iptables can still help prevent brute-force attacks by logging and blocking repeated attempts to login from the same IP address. For example,



```
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name ssh --rsource
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent ! --rcheck --seconds 60 --hitcount 4 --name ssh --rsource -j ACCEPT 
```



The first rule records the IP address of each new attempt to access port 22 using the `recent` module. The second rule checks to see if that IP address has attempted to connect 4 or more times within the last 60 seconds, and if not then the packet is accepted. Note this rule would require a default policy of DROP on the input chain.

Don't forget to change the port as appropriate if you are running ssh on a non-standard port. Where possible, filtering at the firewall is an extremely effective method of securing access to an ssh server.

For systems using the FirewallD service, there is firewall-cmd:

```
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT_direct 0 -p tcp --dport 22 -m state --state NEW -m recent --set
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT_direct 1 -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 30 --hitcount 4 -j REJECT --reject-with tcp-reset
firewall-cmd --reload 
```







## Use Public/Private Keys for Authentication



Using encrypted keys for authentication offers two main benefits. Firstly, it is convenient as you no longer need to enter a password (unless you encrypt your keys with password protection) if you use public/private keys. Secondly, once public/private key pair authentication has been set up on the server, you can disable password authentication completely meaning that without an authorized key you can't gain access - so no more password cracking attempts.

It's a relatively simple process to create a public/private key pair and install them for use on your ssh server.

First, create a public/private key pair on the client that you will use to connect to the server (you will need to do this from each client machine from which you connect):



```
$ ssh-keygen -t rsa
```



This will create two files in your (hidden) **~/.ssh** directory called: **id_rsa** and **id_rsa.pub** The first: **id_rsa** is your private key and the other: **id_rsa.pub** is your public key.

If you don't want to still be asked for a passphrase (which is basically a password to unlock a given public key) each time you connect, just press **enter** when asked for a passphrase when creating the key pair. It is up to you to decide whether or not you should add the passphrase protective encryption to your key when you create it. If you don't passphrase protect your key, then anyone gaining access to your local machine will automatically have ssh access to the remote server. Also, root on the local machine has access to your keys although one assumes that if you can't trust root (or root is compromised) then you're in real trouble. Encrypting the key adds additional security at the expense of eliminating the need for entering a password for the ssh server only to be replaced with entering a passphrase for the use of the key. This may be further simplified by the use of the **ssh_agent** program

Now set permissions on your private key:



```
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/id_rsa 
```



Copy the public key (id_rsa.pub) to the server and install it to the authorized_keys list:



```
$ cat id_rsa.pub >> ~/.ssh/authorized_keys
```



**Note:** once you've imported the public key, you can delete it from the server.

and finally set file permissions on the server:



```
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys 
```



The above permissions are *required* if `StrictModes` is set to yes in **/etc/ssh/sshd_config** (the default).

Ensure the correct SELinux contexts are set:



```
$ restorecon -Rv ~/.ssh 
```



Now when you login to the server you won't be prompted for a password (unless you entered a passphrase when you created your key pair). By default, ssh will first try to authenticate using keys. If no keys are found or authentication fails, then ssh will fall back to conventional password authentication.

Once you've checked you can successfully login to the server using your public/private key pair, you can disable password authentication completely by adding the following setting to your **/etc/ssh/sshd_config** file:



```
# Disable password authentication forcing use of keys
PasswordAuthentication no
```







## Frequently Asked Question (FAQ)



Q: CentOS uses version X of OpenSSH and the latest version is version Y. Version X contained a serious security flaw, should I upgrade?

A: No. The Upstream Vendor has a policy of backporting security patches from the latest releases into the current distribution version. As long as you have the latest updates applied for your CentOS distribution you are fully patched. See here for further details of backporting security patches:

<http://www.redhat.com/advice/speaks_backport.html>

Q: How to I get ssh to allow passwordless key based authentication between machines that share users' home directories by NFS?

A: SElinux blocks root access to NFS shared directories and files that are not world readable by default and so sshd can't read users' key files in ~/.ssh. To enable access, change the setting of use_nfs_home_dirs by running the following command as the superuser:



```
setsebool -P use_nfs_home_dirs 1 
```



<https://www.centos.org/forums/viewtopic.php?t=49194>



## Links



<http://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-openssh.html>

<http://www.dragonresearchgroup.org/insight/sshpwauth-tac.html>



## 其他文章



[加强 CentOS 7 登录安全](https://www.jianshu.com/p/bb0a55e4b11c)

[CentOS7之新建用户与SSH登陆](https://segmentfault.com/a/1190000004141370)

[SSH 公钥验证远程登录](https://xlui.me/t/ssh-public-key-auth/)

