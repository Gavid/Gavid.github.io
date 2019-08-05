* ps进程查询，按照用户分类，查找xxx有关的进程

  > ps -aux |grep xxx

* 根据进程查看进程相关信息占用的内存情况（进程号可以根据上述命令进行查看）

  > pmap -d 进程号

* 杀死指定进程

  > kill -9 进程号

* 将sh test.sh任务放到后台，但是依然可以使用标准输入，终端能够接收任何输入，重定向标准输出和标准错误到当前目录下的nohup.out文件，即使关闭xshell退出当前session依然继续运行。

  > nohup python test.py & 

* 快速清空文件内容

  > &gt; name

