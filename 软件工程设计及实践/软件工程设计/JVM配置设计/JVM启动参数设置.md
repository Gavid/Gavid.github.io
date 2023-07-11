## JVM启动参数

参数兼顾性能及排查问题的便捷性的JVM启动参数推荐， 其中一些参数需要根据JDK版本适配。

源码： [jvm-options](https://github.com/vipshop/vjtools/blob/master/vjstar/src/main/script/jvm-options) 解读：[《关键业务系统的JVM参数推荐》](https://blog.csdn.net/zero__007/article/details/84476930)

---

### jvm-options.sh

```bash
#!/bin/bash

# 使用指南：
# 1. 修改本文件中的LOGDIR 和 APPID变量
# 2. 根据实际情况需求，反注释掉一些参数。
# 3. 修改应用启动脚本，增加 "source ./jvm-options.sh"，或者将本文件内容复制进应用启动脚本里.
# 4. 修改应用启动脚本，使用输出的JAVA_OTPS变量，如java -jar xxx的应用启动语句，修改为 java $JAVA_OPTS -jar xxx。


# change the jvm error log and backup gc log dir here
LOGDIR="./logs"

# change the appid for gc log name here
APPID="myapp"

JAVA_VERSION=$(java -version 2>&1 | awk -F '"' '/version/ {print $2}')


# Enable coredump
ulimit -c unlimited

## Memory Options##

MEM_OPTS="-Xms4g -Xmx4g -XX:NewRatio=1"

if [[ "$JAVA_VERSION" < "1.8" ]]; then
  MEM_OPTS="$MEM_OPTS -XX:PermSize=128m -XX:MaxPermSize=512m"
else         
  MEM_OPTS="$MEM_OPTS -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m"
fi

# 启动时预申请内存
MEM_OPTS="$MEM_OPTS -XX:+AlwaysPreTouch"

# 如果线程数较多，函数的递归较少，线程栈内存可以调小节约内存，默认1M。
#MEM_OPTS="$MEM_OPTS -Xss256k"

# 堆外内存的最大值默认约等于堆大小，可以显式将其设小，获得一个比较清晰的内存总量预估
#MEM_OPTS="$MEM_OPTS -XX:MaxDirectMemorySize=2g"

# 根据JMX/VJTop的观察，调整二进制代码区大小避免满了之后不能再JIT，JDK7/8，是否打开多层编译的默认值都不一样
#MEM_OPTS="$MEM_OPTS -XX:ReservedCodeCacheSize=240M"


## GC Options##

GC_OPTS="-XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly"

# System.gc() 使用CMS算法
GC_OPTS="$GC_OPTS -XX:+ExplicitGCInvokesConcurrent"

# CMS中的下列阶段并发执行
GC_OPTS="$GC_OPTS -XX:+ParallelRefProcEnabled -XX:+CMSParallelInitialMarkEnabled"

# 根据应用的对象生命周期设定，减少事实上的老生代对象在新生代停留时间，加快YGC速度
GC_OPTS="$GC_OPTS -XX:MaxTenuringThreshold=3"

# 如果OldGen较大，加大YGC时扫描OldGen关联的卡片，加快YGC速度，默认值256较低
GC_OPTS="$GC_OPTS -XX:+UnlockDiagnosticVMOptions -XX:ParGCCardsPerStrideChunk=1024"

# 如果JVM并不独占机器，机器上有其他较繁忙的进程在运行，将GC线程数设置得比默认值(CPU核数＊5/8 )更低以减少竞争，反而会大大加快YGC速度。
# 另建议CMS GC线程数简单改为YGC线程数一半.
#GC_OPTS="$GC_OPTS -XX:ParallelGCThreads=12 -XX:ConcGCThreads=6"

# 如果CMS GC时间很长，并且明显受新生代存活对象数量影响时打开，但会导致每次CMS GC与一次YGC连在一起执行，加大了事实上JVM停顿的时间。
#GC_OPTS="$GC_OPTS -XX:+CMSScavengeBeforeRemark"

# 如果永久代使用不会增长，关闭CMS时ClassUnloading，降低CMS GC时出现缓慢的几率
#if [[ "$JAVA_VERSION" > "1.8" ]]; then     
#  GC_OPTS="$GC_OPTS -XX:-CMSClassUnloadingEnabled"
#fi


## GC log Options, only for JDK7/JDK8 ##

# 默认使用/dev/shm 内存文件系统避免在高IO场景下写GC日志时被阻塞导致STW时间延长
if [ -d /dev/shm/ ]; then
    GC_LOG_FILE=/dev/shm/gc-${APPID}.log
else
    GC_LOG_FILE=${LOGDIR}/gc-${APPID}.log
fi


if [ -f ${GC_LOG_FILE} ]; then
  GC_LOG_BACKUP=${LOGDIR}/gc-${APPID}-$(date +'%Y%m%d_%H%M%S').log
  echo "saving gc log ${GC_LOG_FILE} to ${GC_LOG_BACKUP}"
  mv ${GC_LOG_FILE} ${GC_LOG_BACKUP}
fi

#打印GC日志，包括时间戳，晋升老生代失败原因，应用实际停顿时间(含GC及其他原因)
GCLOG_OPTS="-Xloggc:${GC_LOG_FILE} -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintPromotionFailure -XX:+PrintGCApplicationStoppedTime"


#打印GC原因，JDK8默认打开
if [[ "$JAVA_VERSION" < "1.8" ]]; then
    GCLOG_OPTS="$GCLOG_OPTS -XX:+PrintGCCause"
fi


# 打印GC前后的各代大小
#GCLOG_OPTS="$GCLOG_OPTS -XX:+PrintHeapAtGC"

# 打印存活区每段年龄的大小
#GCLOG_OPTS="$GCLOG_OPTS -XX:+PrintTenuringDistribution"

# 如果发生晋升失败，观察老生代的碎片
#GCLOG_OPTS="$GCLOG_OPTS -XX:+UnlockDiagnosticVMOptions -XX:PrintFLSStatistics=2"


# 打印安全点日志，找出GC日志里非GC的停顿的原因
#GCLOG_OPTS="$GCLOG_OPTS -XX:+PrintSafepointStatistics -XX:PrintSafepointStatisticsCount=1 -XX:+UnlockDiagnosticVMOptions -XX:-DisplayVMOutput -XX:+LogVMOutput -XX:LogFile=/dev/shm/vm-${APPID}.log"


## Optimization Options##

OPTIMIZE_OPTS="-XX:-UseBiasedLocking -XX:AutoBoxCacheMax=20000 -Djava.security.egd=file:/dev/./urandom"


# 关闭PerfData写入，避免高IO场景GC时因为写PerfData文件被阻塞，但会使得jstats，jps不能使用。
#OPTIMIZE_OPTS="$OPTIMIZE_OPTS -XX:+PerfDisableSharedMem"

# 关闭多层编译，减少应用刚启动时的JIT导致的可能超时，以及避免部分函数C1编译后最终没被C2编译。 但导致函数没有被初始C1编译。
#if [[ "$JAVA_VERSION" > "1.8" ]]; then     
#  OPTIMIZE_OPTS="$OPTIMIZE_OPTS -XX:-TieredCompilation"
#fi

# 如果希望无论函数的热度如何，最终JIT所有函数，关闭GC时将函数调用次数减半。
#OPTIMIZE_OPTS="$OPTIMIZE_OPTS -XX:-UseCounterDecay"

## Trouble shooting Options##

SHOOTING_OPTS="-XX:+PrintCommandLineFlags -XX:-OmitStackTraceInFastThrow -XX:ErrorFile=${LOGDIR}/hs_err_%p.log"


# OOM 时进行HeapDump，但此时会产生较高的连续IO，如果是容器环境，有可能会影响他的容器
#SHOOTING_OPTS="$SHOOTING_OPTS -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${LOGDIR}/"

# async-profiler 火焰图效果更好的参数
#SHOOTING_OPTS="$SHOOTING_OPTS -XX:+UnlockDiagnosticVMOptions -XX:+DebugNonSafepoints"

# 在非生产环境，打开JFR进行性能记录（生产环境要收License的哈）
#SHOOTING_OPTS="$SHOOTING_OPTS -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:+UnlockDiagnosticVMOptions -XX:+DebugNonSafepoints"



## JMX Options##

#开放JMX本地访问，设定端口号
JMX_OPTS="-Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote.port=7001 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"


## Other Options##

OTHER_OPTS="-Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8"


## All together ##

export JAVA_OPTS="$MEM_OPTS $GC_OPTS $GCLOG_OPTS $OPTIMIZE_OPTS $SHOOTING_OPTS $JMX_OPTS $OTHER_OPTS"

echo JAVA_OPTS=$JAVA_OPTS
```

---

### 《关键业务系统的JVM参数推荐》

#### 1. 取消偏向锁 -XX:-UseBiasedLocking

  JDK1.6开始默认打开的偏向锁，会尝试把锁赋给第一个访问它的线程，取消同步块上的synchronized原语。如果始终只有一条线程在访问它，就成功略过同步操作以获得性能提升。
  但一旦有第二条线程访问这把锁，JVM就要撤销偏向锁恢复到未锁定线程的状态，如果打开安全点日志，可以看到不少RevokeBiasd的纪录，像GC一样Stop The World的干活，虽然只是很短的停顿，但对于多线程并发的应用，取消掉它反而有性能的提升，所以Cassandra就取消了它。

#### 2. 加大Integer Cache -XX:AutoBoxCacheMax=20000

  `Integer i=3;`这语句有着 int自动装箱成Integer的过程，JDK默认只缓存 `-128 ~ +127`的Integer 和 Long，超出范围的数字就要即时构建新的Integer对象。设为20000后，我们应用的QPS有足足4%的影响。

#### 3. 启动时访问并置零内存页面 -XX:+AlwaysPreTouch

  启动时就把参数里说好了的内存全部舔一遍，可能令得启动时慢上一点，但后面访问时会更流畅，比如页面会连续分配，比如不会在晋升新生代到老生代时才去访问页面使得GC停顿时间加长。ElasticSearch和Cassandra都打开了它。

#### 4. -XX:-UseCounterDecay

  禁止JIT调用计数器衰减。默认情况下，每次GC时会对调用计数器进行砍半的操作，导致有些方法一直温热，永远都达不到触发C2编译的1万次的阀值。

#### 3. -XX:-TieredCompilation

  多层编译是JDK8后默认打开的比较骄傲的功能，先以C1静态编译，采样足够后C2编译。
  但我们实测，性能最终略降2%，可能是因为有些方法C1编译后C2不再编译了。应用启动时的偶发服务超时也多了，可能是忙于编译。所以我们将它禁止了，但记得打开前面的-XX:-UseCounterDecay，避免有些温热的方法永远都要解释执行。

### 内存篇

  为了稳健，还是8G以下的堆还是CMS好了，G1现在虽然是默认了，但其实在小堆里的表现也没有比CMS好，还是JDK11的ZGC引人期待。

#### 1. CMS基本写法

  `-XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly`。为了让`CMSInitiatingOccupancyFraction`这个设置生效，还要设置`-XX:+UseCMSInitiatingOccupancyOnly`，否则75％只被用来做开始的参考值，后面还是JVM自己算。

#### 2. -XX:MaxTenuringThreshold=2

  这是改动效果最明显的一个参数了。对象在Survivor区最多熬过多少次Young GC后晋升到年老代，JDK8里CMS 默认是6，其他如G1是15。
  Young GC是最大的应用停顿来源，而新生代里GC后存活对象的多少又直接影响停顿的时间，所以如果清楚Young GC的执行频率和应用里大部分临时对象的最长生命周期，可以把它设的更短一点，让其实不是临时对象的新生代对象赶紧晋升到年老代，别呆着。
  用`-XX:+PrintTenuringDistribution`观察下，如果后面几代的大小总是差不多，证明过了某个年龄后的对象总能晋升到老生代，就可以把晋升阈值设小，比如JMeter里2就足够了。

#### 3. -XX:+ExplicitGCInvokesConcurrent 但不要-XX:+DisableExplicitGC

  full gc时，使用CMS算法，不是全程停顿，必选。

#### 4. ParallelRefProcEnabled 和 CMSParallelInitialMarkEnabled

  并行的处理Reference对象，如WeakReference，默认为false，除非在GC log里出现Reference处理时间较长的日志，否则效果不会很明显，但我们总是要JVM尽量的并行，所以设了也就设了。同理还有`-XX:+CMSParallelInitialMarkEnabled`，JDK8已默认开启，但小版本比较低的JDK7甚至不支持。

#### 5. ParGCCardsPerStrideChunk

  Linkined的黑科技，有些场景的确能减少YGC时间，简单说就是影响YGC时扫描老生代的时间，默认值256太小了，但32K也未必对，需要自己试验。
  ​`-XX:+UnlockDiagnosticVMOptions -XX: ParGCCardsPerStrideChunk=1024`

#### 6. 并发收集线程数

```
ParallelGCThreads＝8+( Processor - 8 ) ( 5/8 )；
ConcGCThreads = (ParallelGCThreads + 3)/4
12
```

  比如双CPU，六核，超线程就是24个处理器，小于8个处理器时ParallelGCThreads按处理器数量，大于时按上述公式YGC线程数＝18， CMS GC线程数＝5。
  ​CMS GC线程数的公式太怪，也有人提议简单改为YGC线程数的1/2。一些不在乎停顿时间的后台辅助程序，比如日志收集的logstash，建议把它减少到2，避免在GC时突然占用太多CPU核，影响主应用。
  ​而另一些并不独占服务器的应用，比如旁边跑着一堆sidecar的，也建议减少YGC线程数。
  ​一个真实的案例，24核的服务器，默认18条YGC线程，但因为旁边有个繁忙的Service Mesh Proxy在跑着，这18条线程并不能100%的抢到CPU，出现了不合理的慢GC。把线程数降低到12条之后，YGC反而快了很多。 所以那些贪心的把YGC线程数＝CPU 核数的，通常弄巧成拙。

#### 7. -XX:－CMSClassUnloadingEnabled

  在CMS中清理永久代中的过期的Class而不等到Full GC，JDK7默认关闭而JDK8打开。看自己情况，比如有没有运行动态语言脚本如Groovy产生大量的临时类。它有时会大大增加CMS的暂停时间。所以如果新类加载并不频繁，这个参数还是显式关闭的好。

#### 8. -XX:+CMSScavengeBeforeRemark

  默认为关闭，在CMS remark前，先执行一次minor GC将新生代清掉，这样从老生代的对象引用到的新生代对象的个数就少了，停止全世界的CMS remark阶段就短一些。如果打开了，会让一次YGC紧接着一次CMS GC，使得停顿的总时间加长了。
  ​又一个真实案例，CMS GC的时间和当时新生代的大小成比例，新生代很小时很快完成，新生代80％时CMS GC停顿时间超过一秒，这时候就还是打开了划算。

#### 9. -XX:CMSFullGCsBeforeCompaction

  默认为0，即每次full gc都对老生代进行碎片整理压缩。Full GC 不同于 老生代75%时触发的CMS GC，只在老生代达到100%，老生代碎片过大无法分配空间给新晋升的大对象，堆外内存满，这些特殊情况里发生，所以设为每次都进行碎片整理是合适的，详见[此贴里R大的解释](http://hllvm.group.iteye.com/group/topic/28854)。

#### 10. -Xverify:none

  来自优化Eclipse启动速度的经验，说关闭Java类加载验证可以加快10% -15%的启动速度。

### 监控篇

#### 1. -XX:+PrintCommandLineFlags

  运维有时会对启动参数做一些临时的更改，将每次启动的参数输出到stdout，将来有据可查。
  ​打印出来的是命令行里设置了的参数以及因为这些参数隐式影响的参数，比如开了CMS后，-XX:+UseParNewGC也被自动打开。

#### 2. -XX:-OmitStackTraceInFastThrow

  为异常设置StackTrace是个昂贵的操作，所以当应用在相同地方抛出相同的异常N次(两万?)之后，JVM会对某些特定异常如NPE，数组越界等进行优化，不再带上异常栈。此时，你可能会看到日志里一条条Nul Point Exception，而之前输出完整栈的日志早被滚动到不知哪里去了，也就完全不知道这NPE发生在什么地方，欲哭无泪。 所以，将它禁止吧，ElasticSearch也这样干。

#### 3. -XX:ErrorFile

  JVM crash时，hotspot 会生成一个error文件，提供JVM状态信息的细节。如前所述，将其输出到固定目录，避免到时会到处找这文件。文件名中的%p会被自动替换为应用的PID:`-XX:ErrorFile=${MYLOGDIR}/hs_err_%p.log`

#### 4. -XX:+HeapDumpOnOutOfMemoryError(可选)

  在Out Of Memory，JVM快死掉的时候，输出Heap Dump到指定文件。不然开发很多时候还真不知道怎么重现错误。
  ​路径只指向目录，JVM会保持文件名的唯一性，叫`java_pid${pid}.hprof`。因为如果指向文件，而文件已存在，反而不能写入。
`-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${LOGDIR}/`
  ​但在容器环境下，输出4G的HeapDump，在普通硬盘上会造成20秒以上的硬盘IO跑满，也是个十足的恶邻，影响了同一宿主机上所有其他的容器。

#### 5. `-Xloggc:/dev/shm/gc-myapp.log -XX:+PrintGCDateStamps -XX:+PrintGCDetails`

  有人担心写GC日志会影响性能，但测试下来实在没什么影响，GC问题是Java里最常见的问题，没日志怎么行。
  ​后来又发现如果遇上高IO的情况，GC时操作系统正在flush pageCache 到磁盘，也可能导致GC log文件被锁住，从而让GC结束不了。所以把它指向了/dev/shm 这种内存中文件系统，避免这种停顿。
  ​用PrintGCDateStamps而不是PrintGCTimeStamps，打印可读的日期而不是时间戳。

#### 6. -XX:+PrintGCApplicationStoppedTime

  这是个非常非常重要的参数，但它的名字没起好，其实除了打印清晰的完整的GC停顿时间外，还可以打印其他的JVM停顿时间，比如取消偏向锁，class 被agent redefine，code deoptimization等等，有助于发现一些原来没想到的问题。如果真的发现了一些不知是什么的停顿，需要打印安全点日志找原因。

#### 7. -XX:+PrintGCCause

  打印产生GC的原因，比如AllocationFailure什么的，在JDK8已默认打开，JDK7要显式打开一下。

#### 8. -XX:+PrintPromotionFailure

  打开了就知道是多大的新生代对象晋升到老生代失败从而引发Full GC时的。

#### 9. GC日志滚动与备份

  GC日志默认会在重启后清空，有人担心长期运行的应用会把文件弄得很大，所以`-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=1M`的参数可以让日志滚动起来。但真正用起来重启后的文件名太混乱太让人头痛，GC日志再大也达不到哪里去，所以我们没有加滚动，而且自行在启动脚本里对旧日志做备份。

#### 10. JMX

```
-Dcom.sun.management.jmxremote.port=7001 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=127.0.0.1
1
```

  以上设置，只让本地的Zabbix之类监控软件通过JMX监控JVM，不允许远程访问。