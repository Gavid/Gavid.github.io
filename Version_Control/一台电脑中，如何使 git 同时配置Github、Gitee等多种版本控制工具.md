@[TOC](一台电脑中，如何使 git 同时配置Github、Gitee等多种版本控制工具)
> 最近由于频繁使用git从Github、Gitee(码云)等代码仓库clone别人的优秀code，但是发现从Github上clone完项目后，想要再从Gitee上clone项目就必须重新配置ssh-key。这样来回生成ssh-key然后还得配置也太麻烦了，接下来，本文章会介绍如何解决这种问题。
# 1. ssh文件本地环境配置
* 在自己系统中找到 ==.ssh==文件夹,**将此目录下的文件全部删除**。
	* 方法一： 在GUI模式下手动寻找 （window系统在 “C:\Users\账号名\\.ssh ” ）
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019071711052845.png)
 	* 方法二： 鼠标右键打开 Git Bash，在弹出的窗口中输入：
 	```console
 	cd ~/.ssh       # 进入.ssh文件夹
 	```
 	![在这里插入图片描述](https://img-blog.csdnimg.cn/20190717110439302.png)
 * 生成ssh配置文件
 	* 在 .ssh 文件夹下鼠标右键打开 Git Base Here 
 	* 输入命令：
 		```console
 		ssh-keygen -t rsa -C "xxxxx@xxxxx.com"    # 填写自己Github / Gitee的邮箱
 		```
 	* 上述命令的执行次数由你要绑定几个代码仓库有关（eg: 要同时绑定Github和Gitee，则上述命令需执行两次），执行的时候可以自己指定生成文件的文件名（默认是id_rsa），然后填写密码（可以为空）：
 		```console
 		Generating public/private rsa key pair.
 		Enter file in which to save the key (C:/Users/jiaha/.ssh): github_rsa
 		```
 		 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190717105307589.png)
 * 创建config文件，写入一些相应配置
 	```console
 	# gitee
	Host gitee.com
	HostName gitee.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_rsa.gitee

	# github
	Host github.com
	HostName github.com
	PreferredAuthentications publickey
	IdentityFile ~/.ssh/id_rsa.github
 	```
# 2. 在Github、Gitee上配置SSH keys（以Github为例）
* 登录自己的Github / Gitee 网站，点击网站右上角自己的头像， 点击 Settings 选项，打开如下窗口，点击"New SSH key"。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190717134149744.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
* “文本框1”中可以自己填写一个自定义标识（可以随便填写），“文本框2”中将之前在本地生成的**_rsa.pub文件中的内容复制粘贴到这里即可。然后点击“Add SSH key”。
	![在这里插入图片描述](https://img-blog.csdnimg.cn/201907171344244.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjIzMDU1MA==,size_16,color_FFFFFF,t_70)
# 3.测试是否成功
* 打开 git 的控制窗口，输入如下命令：
	```console
	# 测试连接 Github 
	ssh -T git@github.com
	# 测试连接 Gitee 
	ssh -T git@gitee.com
	```
* 如果出现如下结果，则说明你已经配置成功了。
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20190717135314964.png)
	参考文献：
	https://blog.csdn.net/qq_33858250/article/details/81046316
	https://my.oschina.net/u/3552749/blog/1678082
