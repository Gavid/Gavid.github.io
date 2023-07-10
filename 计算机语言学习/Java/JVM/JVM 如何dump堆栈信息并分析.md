### JVM 如何dump堆栈信息并分析

dump服务的堆栈信息主要有两种渠道，分别是自动生成dump文件和手动生成dump文件。

一、自动生成dump文件

顾名思义，当程序中出现内存溢出或者gc的时候系统会自动dump出堆栈信息到指定路径文件中。

为了实现上述功能需要在jvm启动参数中添加如下的配置

```bash
# 1. 当OutOfMemoryError发生时自动生成 Heap Dump 文件。(当你需要分析Java内存使用情况时，往往是在OOM发生时)
-XX:+HeapDumpOnOutOfMemoryError
# 2. 当 JVM 执行 FullGC 前执行 dump。 
-XX:+HeapDumpBeforeFullGC
# 3. 当 JVM 执行 FullGC 后执行 dump。
-XX:+HeapDumpAfterFullGC
# 4. 交互式获取dump。在控制台按下快捷键Ctrl + Break时，JVM就会转存一下堆快照。
-XX:+HeapDumpOnCtrlBreak
# 5. 指定 dump 文件的存储路径
-XX:HeapDumpPath=/opt/logs/dumplogs
```

二、手动生成dump文件，主要的步骤如下：

```bash
# 1.登录上有问题的机器并获取到相应的root权限。
# 2.通过top指令知道需要dump的java进程
# 3.开始dump文件。例如：jmap -dump:format=b,file=202104012.hprof 6666
jmap -dump:file=文件名.hprof,format=b pid（进程号） 
```

使用自动和手动dump文件的方式会把dump的文件存储到对应的服务中需要==将文件下载到本地进行分析==

三、copy dump文件到本地

```bash
# 1.登录自己的dump文件的服务器，执行 sftp xxx@jumper.xxx.cn,成功后执行命令 put 202104012.hprof(dump文件名) 传输文件。
# 2.文件传输完成后在本地shell 输入命令 sftp xxx@jumper.xxx.cn,命令成功后执行get 202104012.hprof(dump文件名) 拉取文件到本地。
```

四、使用工具对dump文件进行分析

dump文件到本地以后就可以使用mat、Jprofile、visualVM等工具进行分析dump文件

1. 使用Mat工具进行分析

   > 1. Mat 工具下载（http://www.eclipse.org/mat/downloads.php）
   >
   > 2. 将mat 工具压缩包解压：
   >
   >    * MemoryAnalyzer.ini 配置文件可以修改最大的内存，默认1G基本够用了。
   >
   > 3. 执行分析命令
   >
   >    `./ParseHeapDump.sh m.hprof  org.eclipse.mat.api:suspects org.eclipse.mat.api:overview org.eclipse.mat.api:top_components`
   >
   >    m.hprof就是jvm的dump文件，在mat目录下会生成3份.zip结尾的报告和一些m.相关的文件，将生成的m.hprof相关的文件都下载到本地磁盘
   >
   > 4. 查看分析报告
   >
   >    使用浏览器打开index.html文件内容，查看分析报告
   >
   >    主要查看Class Histogram一项（Shallow Heap 既对象本身的大小；Retained Heap 对象自身加起直接或间接引用的大小）

