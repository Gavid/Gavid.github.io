- [easygc.io](http://www.gceasy.io/) gc日志分析

### 如何启用 Java GC 日志记录功能

> 对于 Java1.4、5、6、7、8，将以下 JVM 参数传递给您的应用程序即可： **-XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:<file-path>**
>
> 对于 Java 9，传递以下 JVM 参数： **-Xlog:gc\*:file=<file-path>**
> **file-path:** 写入 GC 日志文件的位置

