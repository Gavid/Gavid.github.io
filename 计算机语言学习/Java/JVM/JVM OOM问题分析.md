### OOM问题分析

各种OOM的情况

```bash
1. 堆溢出-java.lang.OutOfMemoryError: Java heap space。
2. 栈溢出-java.lang.OutOfMemorryError。
3. 栈溢出-java.lang.StackOverFlowError。
4. 元信息溢出-java.lang.OutOfMemoryError: Metaspace。
5. 直接内存溢出-java.lang.OutOfMemoryError: Direct buffer memory。
6. GC超限-java.lang.OutOfMemoryError: GC overhead limit exceeded。
```

分析OOM问题原因

```bash
查看jvm进程：jps
查看jvm堆内存情况：jmap -heap 进程号
查看堆内存中的对象数目：jmap -histo:live 进程号 | more
查看实时cpu、内存情况：top
查看内存：cat /proc/meminfo
 
java -jar -Xms128M -Xmx256M -XX:PermSize=128M -XX:MaxPermSize=256M ***.jar
 
Xms : 堆内存初始大小
Xmx : 堆内存最大值
PermSize : 永久内存初始大小
MaxPermSize ： 永久内存最大值
```



