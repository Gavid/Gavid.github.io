> ==Note: 设置前提——两台机器网络互通(即可ping通)==

## 一、概述

1、就是为了让两个linux机器之间使用ssh不需要用户名和密码。采用了数字签名完成这个操作

2、模型分析

假设 A （192.168.20.59）为客户机器，B（192.168.20.60）为目标机；

要达到的目的：
A机器ssh登录B机器无需输入密码；
加密方式可参考[生成新SSH密钥](https://docs.github.com/zh/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)

## 二、具体操作流程 

#### 单向登陆的操作过程（能满足上边的目的）：

1、登录A机器
2、ssh-keygen -t [rsa|dsa|ed25519]，将会生成密钥文件和私钥文件
3、将 .pub 文件复制到B机器的 .ssh 目录， 并 cat id_dsa.pub >> ~/.ssh/authorized_keys
4、大功告成，从A机器登录B机器的目标账户，不再需要密码了；（直接运行 **#ssh 192.168.20.60** ）

> 设置好后，首次`ssh 192.168.20.60` 需要手动输入 `yes`,确认链接
>
> 若不想手动输入确认（批量操作场景），可以使用 `ssh -o StrictHostKeyChecking=no 192.168.20.60`，即可跳过确认流程。

---

#### 双向登陆的操作过程：

1、ssh-keygen做密码验证可以使在向对方机器上ssh ,scp不用使用密码.具体方法如下:
2、两个节点都执行操作：**#ssh-keygen -t [rsa|dsa|ed25519]**
 然后全部回车,采用默认值.

3、这样生成了一对密钥，存放在用户目录的~/.ssh下。
**将公钥考到对方机器的用户目录下**，并将其复制到~/.ssh/authorized_keys中（操作命令：**#cat id_dsa.pub >> ~/.ssh/authorized_keys**）。



4、设置文件和目录权限：

设置authorized_keys权限
`chmod 600 authorized_keys`
设置.ssh目录权限
` chmod 700 -R .ssh`

 

5、要保证.ssh和authorized_keys都只有用户自己有写权限。否则验证无效。（今天就是遇到这个问题，找了好久问题所在），其实仔细想想，这样做是为了不会出现系统漏洞。

我从20.60去访问20.59的时候会提示如下错误：

**[java]** [view plain](http://blog.csdn.net/kongqz/article/details/6338690#) [copy](http://blog.csdn.net/kongqz/article/details/6338690#)

1. The authenticity of host '192.168.20.59 (192.168.20.59)' can't be established. 
2. RSA key fingerprint is 6a:37:c0:e1:09:a4:29:8d:68:d0:ca:21:20:94:be:18. 
3. Are you sure you want to **continue** connecting (yes/no)? yes 
4. Warning: Permanently added '192.168.20.59' (RSA) to the list of known hosts. 
5. root@192.168.20.59's password:  
6. Permission denied, please **try** again. 
7. root@192.168.20.59's password:  
8. Permission denied, please **try** again. 
9. root@192.168.20.59's password:  
10. Permission denied (publickey,gssapi-with-mic,password). 

## 三、总结注意事项

1、文件和目录的权限千万别设置成chmod 777.这个权限太大了，不安全，数字签名也不支持。我开始图省事就这么干了

2、生成的rsa/dsa签名的公钥是给对方机器使用的。这个公钥内容还要拷贝到authorized_keys

3、linux之间的访问直接 ssh 机器ip

4、某个机器生成自己的RSA或者DSA的数字签名，将公钥给目标机器，然后目标机器接收后设定相关权限（公钥和authorized_keys权限），这个目标机就能被生成数字签名的机器无密码访问了