@[TOC](Linux下，查看硬件信息命令收集)

### 查看cpu的命令 

```bash
dmesg |grep -i xeon
```

### 查看内存的命令

```bash
free -m
```

### 查看硬盘的命令

```bash
df -h
```

### 查看系统版本信息

```bash
head -n 1 /etc/issue
```

### 查看系统安装的所有软件

```bash
dpkg -l
```

### 查看xxx软件的安装路径

```bash
whereis xxx
```

### LINUX中如何查看某个端口是否被占用的方法

```bash
# 主要看监控状态为LISTEN表示已经被占用，最后一列显示被xxx服务占用
netstat -anp | grep 端口号

# 该命令是查看当前所有已经使用的端口情况
netstat -nultp（此处不用加端口号）

netstat -anp |grep 82 # 查看82端口的使用情况
# 此处注意，图中显示的LISTENING并不表示端口被占用，不要和LISTEN混淆哦，查看具体端口时候，必须要看到tcp，端口号，LISTEN那一行，才表示端口被占用了
```

### /bin/bash -c的作用 :  让 bash 将一个字符串作为完整的命令来执行

```bash
 /bin/bash -c 'echo "kettle" >> nohup.log'
```

### strings /proc/{进程号}/environ : 查看进程环境变量信息

### jstack查找高度占用CPU的java代码

```bash
top -- 找到最耗CPU的进程号 pid  P：按%CPU使用率排行 M：按%MEM排行
ps -mp pid -o THREAD,tid -- 找到最耗CPU的java线程
printf "%x\n" pid  -- 打印出的是十六进制
jstack pid | grep 十六进制 -C11 --color
```

### 「tail head」查看文档的指定位置内容

```bash
【一】从第3000行开始，显示1000行。即显示3000~3999行
cat filename | tail -n +3000 | head -n 1000
【二】显示1000行到3000行
cat filename| head -n 3000 | tail -n +1000 
*注意两种方法的顺序
分解：
    tail -n 1000：显示最后1000行
    tail -n +1000：从1000行开始显示，显示1000行以后的
    head -n 1000：显示前面1000行
【三】用sed命令
 sed -n '5,10p' filename 这样你就可以只查看文件的第5行到第10行。
```

### 「 > 」将结果输出到某个文件中 

```bash
# 清空文件后，从首部重写文件内容
> filename
# 追加文件内容
>> filename
```

### 「 wc 」统计

```bash
# 统计有多少行
| wc -l
```

### 「grep」

```bash
-v --revert-match # 反转查找。

-a --text  # 不要忽略二进制数据。
-A <显示行数>   --after-context=<显示行数>   # 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
-b --byte-offset                           # 在显示符合范本样式的那一行之外，并显示该行之前的内容。
-B<显示行数>   --before-context=<显示行数>   # 除了显示符合样式的那一行之外，并显示该行之前的内容。
-c --count    # 计算符合范本样式的列数。
-C<显示行数> --context=<显示行数>或-<显示行数> # 除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
-d<进行动作> --directories=<动作>  # 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作。
-e<范本样式> --regexp=<范本样式>   # 指定字符串作为查找文件内容的范本样式。
-E --extended-regexp             # 将范本样式为延伸的普通表示法来使用，意味着能使用扩展正则表达式。
-f<范本文件> --file=<规则文件>     # 指定范本文件，其内容有一个或多个范本样式，让grep查找符合范本条件的文件内容，格式为每一列的范本样式。
-F --fixed-regexp   # 将范本样式视为固定字符串的列表。
-G --basic-regexp   # 将范本样式视为普通的表示法来使用。
-h --no-filename    # 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。
-H --with-filename  # 在显示符合范本样式的那一列之前，标示该列的文件名称。
-i --ignore-case    # 忽略字符大小写的差别。
-l --file-with-matches   # 列出文件内容符合指定的范本样式的文件名称。
-L --files-without-match # 列出文件内容不符合指定的范本样式的文件名称。
-n --line-number         # 在显示符合范本样式的那一列之前，标示出该列的编号。
-P --perl-regexp         # PATTERN 是一个 Perl 正则表达式
-q --quiet或--silent     # 不显示任何信息。
-R/-r  --recursive       # 此参数的效果和指定“-d recurse”参数相同。
-s --no-messages  # 不显示错误信息。
-V --version      # 显示版本信息。   
-w --word-regexp  # 只显示全字符合的列。
-x --line-regexp  # 只显示全列符合的列。
-y # 此参数效果跟“-i”相同。
-o # 只输出文件中匹配到的部分。
-m <num> --max-count=<num> # 找到num行结果后停止查找，用来限制匹配行数
```

### 「df」用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为KB。

```bash
# 语法
# df(选项)(参数)

# 选项
-h或--human-readable：以可读性较高的方式来显示信息；

-a或--all：包含全部的文件系统；
--block-size=<区块大小>：以指定的区块大小来显示区块数目；
-H或--si：与-h参数相同，但在计算时是以1000 Bytes为换算单位而非1024 Bytes；
-i或--inodes：显示inode的信息；
-k或--kilobytes：指定区块大小为1024字节；
-l或--local：仅显示本地端的文件系统；
-m或--megabytes：指定区块大小为1048576字节；
--no-sync：在取得磁盘使用信息前，不要执行sync指令，此为预设值；
-P或--portability：使用POSIX的输出格式；
--sync：在取得磁盘使用信息前，先执行sync指令；
-t<文件系统类型>或--type=<文件系统类型>：仅显示指定文件系统类型的磁盘信息；
-T或--print-type：显示文件系统的类型；
-x<文件系统类型>或--exclude-type=<文件系统类型>：不要显示指定文件系统类型的磁盘信息；
--help：显示帮助；
--version：显示版本信息。

# 参数
文件：指定文件系统上的文件。

# 常用使用方式：
df -h /*
df -hl 查看磁盘剩余空间
```

### 「du」查看磁盘中某个目录文件下“目录及目录下所包含文件的磁盘占用量”

```bash
# 语法
du [选项][文件]

# 选项
-a, --all                              显示目录中个别文件的大小。
-B, --block-size=大小                  使用指定字节数的块
-b, --bytes                            显示目录或文件大小时，以byte为单位。
-c, --total                            除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
-D, --dereference-args                 显示指定符号链接的源文件大小。
-H, --si                               与-h参数相同，但是K，M，G是以1000为换算单位。
-h, --human-readable                   以K，M，G为单位，提高信息的可读性。
-k, --kilobytes                        以KB(1024bytes)为单位输出。
-l, --count-links                      重复计算硬件链接的文件。
-m, --megabytes                        以MB为单位输出。
-L<符号链接>, --dereference<符号链接>  显示选项中所指定符号链接的源文件大小。
-P, --no-dereference                   不跟随任何符号链接(默认)
-0, --null                             将每个空行视作0 字节而非换行符
-S, --separate-dirs                    显示个别目录的大小时，并不含其子目录的大小。
-s, --summarize                        仅显示总计，只列出最后加总的值。
-x, --one-file-xystem                  以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
-X<文件>, --exclude-from=<文件>        在<文件>指定目录或文件。
--apparent-size                        显示表面用量，而并非是磁盘用量；虽然表面用量通常会小一些，但有时它会因为稀疏文件间的"洞"、内部碎片、非直接引用的块等原因而变大。
--files0-from=F                        计算文件F中以NUL结尾的文件名对应占用的磁盘空间如果F的值是"-"，则从标准输入读入文件名
--exclude=<目录或文件>                 略过指定的目录或文件。
--max-depth=N                          显示目录总计(与--all 一起使用计算文件)当N为指定数值时计算深度为N，等于0时等同--summarize
--si                                   类似-h，但在计算时使用1000 为基底而非1024
--time                                 显示目录或该目录子目录下所有文件的最后修改时间
--time=WORD                            显示WORD时间，而非修改时间：atime，access，use，ctime 或status
--time-style=样式                      按照指定样式显示时间(样式解释规则同"date"命令)：full-iso，long-iso，iso，+FORMAT
--help                                 显示此帮助信息并退出
--version                              显示版本信息并退出

# 常用使用方式：
du -sh /opt/*

du -sh [目录名] 返回该目录的大小

du -sm [文件夹] 返回该文件夹总M数

du -h [目录名] 查看指定文件夹下的所有文件大小（包含子文件夹）
```

### 「nohup」将命令放到后台执行

```bash
nohup {命令}  >> out.log 2>&1 &
```

### 「crontab」Linux Crontab 定时任务

```bash
crontab [-u username]　　　　//省略用户表表示操作当前用户的crontab
    -e      (编辑工作表)
    -l      (列出工作表里的命令)
    -r      (删除工作作)
    
# 我们用crontab -e进入当前用户的工作表编辑，是常见的vim界面。每行是一条命令。

#crontab的命令构成为 时间+动作，其时间有分、时、日、月、周五种，操作符有

# * 取值范围内的所有数字
# / 每过多少个数字
# - 从X到Z
# ，散列数字

实例1：每1分钟执行一次myCommand
* * * * * myCommand
实例2：每小时的第3和第15分钟执行
3,15 * * * * myCommand
实例3：在上午8点到11点的第3和第15分钟执行
3,15 8-11 * * * myCommand
实例4：每隔两天的上午8点到11点的第3和第15分钟执行
3,15 8-11 */2  *  * myCommand
实例5：每周一上午8点到11点的第3和第15分钟执行
3,15 8-11 * * 1 myCommand
实例6：每晚的21:30重启smb
30 21 * * * /etc/init.d/smb restart
实例7：每月1、10、22日的4 : 45重启smb
45 4 1,10,22 * * /etc/init.d/smb restart
实例8：每周六、周日的1 : 10重启smb
10 1 * * 6,0 /etc/init.d/smb restart
实例9：每天18 : 00至23 : 00之间每隔30分钟重启smb
0,30 18-23 * * * /etc/init.d/smb restart
实例10：每星期六的晚上11 : 00 pm重启smb
0 23 * * 6 /etc/init.d/smb restart
实例11：每一小时重启smb
0 */1 * * * /etc/init.d/smb restart
实例12：晚上11点到早上7点之间，每隔一小时重启smb
0 23-7/1 * * * /etc/init.d/smb restart
```

### 「sed」[linux系统sed命令删除前几个字符、后几个字符及特定字符前后字符](https://www.cnblogs.com/liujiaxin2018/p/14346844.html)

```bash
# 1、测试数据
[root@PC3 test]# cat a.txt
1234567849
1234567849
1234567849
1234567849
# 2、删除前几个字符
[root@PC3 test]# sed 's/..//' a.txt  ## 删除前两个字符
34567849
34567849
34567849
34567849
[root@PC3 test]# sed 's/...//' a.txt ## 删除前三个字符
4567849
4567849
4567849
4567849
[root@PC3 test]# sed 's/.\{3\}//' a.txt ## 删除前三个字符
4567849
4567849
4567849
4567849
[root@PC3 test]# sed 's/.\{5\}//' a.txt  ## 删除前5个字符
67849
67849
67849
67849
# 3、删除后几个字符
[root@PC3 test]# sed 's/.$//' a.txt ## 删除最后一个字符
123456784
123456784
123456784
123456784
[root@PC3 test]# sed 's/..$//' a.txt  ## 删除最后两个字符
12345678
12345678
12345678
12345678
# 4、删除特定字符及其前的字符
[root@PC3 test]# sed 's/.4//' a.txt
12567849
12567849
12567849
12567849
[root@PC3 test]# sed 's/..4//' a.txt
1567849
1567849
1567849
1567849
# 5、删除特定字符及其后的字符
[root@PC3 test]# sed 's/4.//' a.txt
12367849
12367849
12367849
12367849
[root@PC3 test]# sed 's/4..//' a.txt
1237849
1237849
1237849
1237849
```

### 「pwdx」[Linux](https://so.csdn.net/so/search?from=pc_blog_highlight&q=Linux)中的pwdx命令，利用进程号作为参数，可以打印出指定进程号的工作目录，帮助我们区分不同的进程。

```
pwdx <pid>
```

### 「nc」mac 与服务器 之间传输文件

```bash
# 1.在服务器端设置监听
nc -l 9999 >dict
# 2.在本地传输文件
nc 10.252.152.150 9999 <my_ext_words.dic
```

### 「sftp」Mac 与服务器之间传输文件

```bash
# 1.登录sftp服务器
sftp user@sftp.xxx.com

# 2.文件上传端
put /path/filename(本地主机) /path/filename(远端主机)

# 3.文件下载端
get /path/filename(远端主机) /path/filename(本地主机)

# 本地和远端操作命令区别
# 在sftp的环境下的操作就和一般ftp的操作类似了，ls,rm,mkdir,dir,pwd,等指令都是对远端进行操作，如果要对本地操作，只需在上述的指令上加‘l’变为：lls,lcd, lpwd等。
```

### linux查看磁盘剩余空间以及cpu使用情况

```bash
1、查看物理CPU个数
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
1.1 查看逻辑CPU的个数
# 逻辑CPU数量=物理cpu数量 x cpu cores 这个规格值 x 2(如果支持并开启超线程技术[HT])
cat /proc/cpuinfo| grep "processor"| wc -l
top可以实时的查看cpu的使用情况（Linux下top查看的CPU也是逻辑CPU个数）
2、查看CPU核数
cat /proc/cpuinfo | grep "cpu cores" | uniq
3、查看CPU型号
cat /proc/cpuinfo | grep 'model name' |uniq
4、查看内存
cat /proc/meminfo | grep MemTotal
5、查看磁盘空间
fdisk -l //看到的是物理磁盘大小（包括swap分区的物理大小）
df -h //看到的是文件系统使用状况（不包括swap分区）

```

**df 命令：**

linux中df命令的功能是用来检查linux服务器的文件系统的磁盘空间占用情况。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

1．命令格式：

df [选项] [文件]

2．命令功能：

显示指定磁盘文件的可用空间。如果没有文件名被指定，则所有当前被挂载的文件系统的可用空间将被显示。默认情况下，磁盘空间将以 1KB 为单位进行显示，除非环境变量 POSIXLY_CORRECT 被指定，那样将以512字节为单位进行显示。

“df -h”这条命令再熟悉不过。以更易读的方式显示目前磁盘空间和使用情况。

“df -i” 以inode模式来显示磁盘使用情况。

**df -h 和df -i的区别是什么？同样是显示磁盘使用情况，为什么显示占用百分比相差甚远？**

df -h的比较好解释，就是查看磁盘容量的使用情况。

至于df -i，先需要去理解一下inode
以博客主的个人理解，最简单的说法，inode包含的信息：文件的字节数，拥有者id，组id，权限，改动时间，链接数，数据block的位置。相反是不表示文件大小。这就是为什么df -h和df -i 显示的结果是不一样的原因。

[ps](http://www.111cn.net/fw/photo.html)：在df -h 和df -i 显示使用率100%，基本解决方法都是删除文件。

df -h  是去删除比较大无用的文件-----------大文件占用大量的磁盘容量。

df -i  则去删除数量过多的小文件-----------过多的文件占用了大量的inode号。

### cpu.idle

cpu.idle指的是CPU处于空闲状态时间比例，从时间的角度衡量CPU的空闲程度。

CPU利用率主要分为用户态，系统态和空闲态，分别表示CPU处于用户态执行的时间，系统内核执行的时间，和空闲系统进程执行的时间，三者之和就是CPU的总时间。

当没有用户进程、系统进程等需要执行的时候，CPU就执行系统缺省的空闲进程。cpu.idle就是指空闲进程占用时间的比例，即CPU执行空闲的时间 / CPU总的执行时间。cpu.idle是基于/proc/stat计算出来的。

#### cpu.idle过低排查方式

1. top 或者 jps 或者 ps -ef | grep java,假定我们找到的是java 进程 PID=9527
2. 使用命令：top -H -p 9527，找到java进程中的线程情况
3. 找到占用cpu较高的排名靠前的几个线程 PID 假如找到的耗CPU较高的线程 PID=95271
4. 查看该线程的堆栈信息，使用命令：jstack -l 95271 或者 jstack -F 95271
5. 分析堆栈的信息，进一步进行排查代码逻辑

### cpu.load

cpu.load被定义为在特定时间间隔内运行队列中(在CPU上运行或者等待运行多少进程)的平均进程数。如果一个进程满足以下条件则其就会位于运行队列中：

- 它没有在等待I/O操作的结果
- 它没有主动进入等待状态(也就是没有调用’wait’)
- 没有被停止(例如：等待终止)

Linux load averages 可以衡量任务对系统的需求，并且它可能大于系统当前正在处理的数量，大多数工具将其显示为三个平均值，分别为 1、5 和 15 分钟值。

在Linux中，进程分为三种状态，一种是阻塞的进程blocked process，一种是可运行的进程runnable process，另外就是正在运行的进程running process。

进程可运行状态时，它处在一个运行队列run queue中，与其他可运行进程争夺CPU时间。 系统的load是指正在运行和准备好运行的进程的总数。比如现在系统有2个正在运行的进程，3个可运行进程，那么系统的load就是5。cpu.load是基于/proc/loadavg进行统计的。

#### cpu.load评估

如果cpu.load过大则表示有部分线程在等待获得cpu资源，过小表示cpu资源比较空闲。

对于cpu.load多少开始出现性能问题，外界有不同的说法，有的认为cpu.load/cores最好不要超过1，有的认为cpu.load/cores最好不要超过3，有的认为cpu.load不超过2*cores-2即可。

针对技术商服务进行压测时，发现load低时，接口TP999响应时间<50ms，而cpu.load/cores>4时，TP999响应时间约为200ms左右，多出来的时间可以理解为线程等待cpu处理的时间，可见cpu.load/cores过高时是影响接口响应时间的。

因此可以结合预期的接口响应时间，来定义每个服务的cpu.load/cores不超过多少。比如要求接口响应时间尽可能快的，最好确保cpu.load/cores不超过1，而对时间敏感性要求不太高时，一般要求cpu.load/cores不超过3。



### linux 压缩与解压缩命令

* .gz文件

```bash
1. 压缩文件
gzip 源文件
如压缩 b.txt 使用命令 gzip b.txt  注意 压缩为 .gz 文件 源文件会消失
如果想保留源文件 使用命令  gzip -c 源文件 > 压缩文件
2. 压缩目录
gzip -r 目录
注意 gzip 压缩目录 只会压缩目录下的所有文件 不会压缩目录
3. 解压
gzip -d 压缩文件
```

* .zip文件

```bash
1、zip压缩文件
zip命令的参数很多,可以利用"zip --help"查看，在这里就不在一一说明了，下面只说几个常用的

zip -q -r -e -m -o 'yourName.zip'  "zipfile list''
-q ：不显示压缩进度状态
-r ：子目录子文件全部压缩为zip  //不然的话只有"zipfile list''文件夹被压缩，里面内容没有被压缩进去
-e ：压缩文件需要加密，终端会提示你输入密码的 //zip -r -P test password.zip "zipfile list'' 直接用'test'来加密password.zip 。
-m ：压缩完删除原文件
-o ：设置所有被压缩文件的最后修改时间为当前压缩时间

跨目录的时候是这么操作的
zip -q -r -e -m -o '\user\someone\someDir\someFile.zip' '\users\someDir'

2、unzip解压文件
语法：unzip [options] 压缩文件名.zip，具体跟多的参数可以直接执行"unzip"查看

常用options的含义分别为： 
-x ：文件列表解解压缩文件，但不包括指定的file文件。 
-v ：查看压缩文件目录，但不解压。 
-t ：测试文件有无损坏，但不解压。 
-d ：目录 把压缩文件解到指定目录下。 
-z ：只显示压缩文件的注解。 
-n ：不覆盖已经存在的文件。 
-o ：覆盖已存在的文件且不要求用户确认。 
-j ：不重建文档的目录结构，把所有文件解压到同一目录下。 

eg1：将压缩文件text.zip在当前目录下解压缩。 
unzip text.zip 
eg2：将压缩文件text.zip在指定目录/tmp下解压缩，如果已有相同的文件存在，要求unzip命令不覆盖原先的文件。 
unzip -n text.zip -d /tmp
eg3：查看压缩文件目录，但不解压。 
unzip -v text.zip 
eg4：文件列表解压，指定不解压的文件
unzip text.zip -x test
```

* .tar文件

```bash
语法：tar [主选项+辅选项] 文件或者目录 
使用该命令时，主选项是必须要有的，它告诉tar要做什么事情，辅选项是辅助使用的，可以选用。 
主选项：
-c Create  -r Add/Replace  -t List  -u Update  -x Extract

辅选项：
其中辅选项又分打包或解包通用选项和只解包用的选项

通用选项：
  -b # :#为一数字，每个I / O块使用＃字节的记录，默认512
  -f ：存档位置
  -v ：细报告tar处理的文件信息。如无此选项，tar不报告文件信息。 
  -w ：每一步都要求确认
解压常用选项：
 -k：保存已存在的文件不覆盖
  -m ：还原文件时，把所有文件的修改时间设定为现在
  -O ：将条目标准输出，不还原到磁盘
  -p：恢复权限（包括ACL，作者，文件标记）

例1：把/home目录下包括它的子目录全部打包，打包文件名为usr.tar。 
$ tar cvf usr.tar /home 
例2：把/home目录下包括它的子目录全部打包，并进行压缩，文件名为usr.tar.gz 。 
$ tar czvf usr.tar.gz /home 
例3：把压缩文件usr.tar.gz还原并解包。 
$ tar xzvf usr.tar.gz 
例4：查看usr.tar备份文件的内容，并以分屏方式显示在显示器上。 
$ tar tvf usr.tar | more 
要将文件备份到一个特定的设备，只需把设备名作为备份文件名。 
例5：用户在/dev/fd0设备的软盘中创建一个备份文件，并将/home 目录中所有的文件都拷贝到备份文件中。 
$ tar cf /dev/fd0 /home 
要恢复设备磁盘中的文件，可使用xf选项： 
$ tar xf /dev/fd0 
```

* .rar文件

```bash
# rar和unrar命令需要自己安装,可以直接通过brew安装
解压文件：unrar x test.rar
压缩文件A和B：rar a 压缩后.rar A B
```

### 「jq」一个灵活的轻量级命令行JSON处理器

```bash
# 以json的方式输出
cat test.json | jq .
```

### Linux中执行sh文件时提示:nohup: 无法运行命令“./startup.sh“: 权限不够

```bash
chmod u+x *.sh
```

### crontab 使用日期时间命名重定向文件

```bash
# 使用月份命名
0 12 * * * php /Users/fdipzone/test.php >> "/Users/fdipzone/$(date +"\%Y-\%m").log" 2>&1
# 使用周命名
0 12 * * * php /Users/fdipzone/test.php >> "/Users/fdipzone/$(date +"\%Y-W\%W").log" 2>&1
# 使用小时命名
* * * * * php /Users/fdipzone/test.php >> "/Users/fdipzone/$(date +"\%Y-\%m-\%d_\%H").log" 2>&1
```

### tree 树状图列出目录的内容

```bash
tree -I node_modules # 忽略当前目录文件夹node_modules
tree -P node_modules # 列出当前目录文件夹node_modules的目录结构
tree -P node_modules -L 2 # 显示目录node_modules两层的目录树结构
tree -L 2 > /home/www/tree.txt # 当前目录结果存到 tree.txt 文件中
```

