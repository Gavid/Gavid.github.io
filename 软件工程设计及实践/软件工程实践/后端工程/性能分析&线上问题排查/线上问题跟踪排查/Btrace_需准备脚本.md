# [BTrace](http://github.com/btraceio/btrace)

BTrace是神器，每一个需要每天解决线上问题，但完全不用BTrace的Java工程师，都是可疑的 -- 凯尔文. 萧

BTrace的最大好处，是可以通过自己编写的脚本，获取应用的一切调用信息。而不需要不断地修改代码，加入System.out.println()， 然后重启，然后重启，然后重启应用！！！

同时，特别严格的约束，保证自己的消耗特别小，只要定义脚本时不作大死，直接在生产环境打开也没影响。

在网上搜索BTrace出来的文章都有点旧了，而且不够详细，于是决定，重新写一份，这个版本顺便把代码改成可C&P的。

### 1. 概述

#### 1.1 快速开始

BTrace搬家了!! 已经从 Sun 搬到了http://github.com/btraceio/btrace。

在Release页面里下载最新Zip版，解压就能用，UserGuide和Samples也在里面。

先抄一个UserGuide里的例子：

```java
import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.*;

@BTrace
public class HelloWorld {
  @OnMethod(clazz="java.lang.Thread", method="start")
  public static void onThreadStart() {
     println("thread start!");
  }
}
```

然后ps找出要监控的java应用的pid， `./btrace $pid HelloWorld.java`  就跑起来了。

是不是很简单？？基本上不用任何BTrace的知识，都能猜出HelloWorld会干啥。通过JVM Attach API，BTrace把自己绑进了被监控的进程，按HelloWorld.java里的定义，进行AOP式的代码植入。

最开心就是这里，如果还想监控其他内容，直接修改HelloWorld.java，再执行一次btrace就可以了，不需要重启应用!! 重启应用!!



#### 1.2 典型的场景

1. 服务慢，能找出慢在哪一步，哪个函数里么？

2. 谁调用了System.gc()，调用栈为何？

3. 谁构造了一个超大的ArrayList?

4. 什么样的入参或对象属性，导致抛出了这个异常？或进入了这个处理分支？



#### 1.3 一些重要事情

为了避免BTrace脚本的消耗过大影响真正业务，所以定义了一系列不允许的事情：比如只能调用BTraceUtils 里的一系列方法和脚本里定义的static方法，不允许其他调用任何类的任何方法。 比如不允许创建对象，比如不允许For 循环等等，更多规定看User Guide。 

当然，可以用-u 运行在unsafe mode来规避限制，但不推荐。

在以前的例子里，甚至还不能字符串相加，必须用strcat:

`println(strcat(strcat(probeClass, "."), probeMethod));`

好在新版里已经可以写回：

`println(probeClass + '.' + probeMethod);`

另外，BTrace植入过的代码，会一直在，直到应用重启为止。所以即使BTrace退出了，业务函数每次执行时都会多出一次BTrace是否Attach状态的判断。



#### 1.4 其他命令行选项

##### 1.4.1 定义classpath

如果在HelloWorld.java里使用了JDK外的其他类，比如Netty的:

`./btrace -cp .:netty-all-4.0.41.Final.jar $pid HelloWorld.java`

但上面定义的classpath只在编译脚本时使用，而脚本里需要显式使用非JDK类的机会其实很少(后面真正用到的时候会提起)。

而在运行时，因为已经绑到目标应用的JVM里，用的是目标JVM的classpath。 



##### 1.4.2 结果输出到文件

`./btrace -o mylog $pid HelloWorld.java`

很坑新人的参数，首先，这个mylog会生成在应用的启动目录，而不是btrace的启动目录。其次，执行过一次-o之后，再执行btrace不加-o 也不会再输出回console，直到应用重启为止。

所以最好直接用转向了事：
`./btrace $pid HelloWorld.java > mylog`

 

##### 1.4.3.预编译脚本

虽然btrace可以实时编译Java源文件，但如果你的脚本是要给运维同学执行的，线上运行时才发现写错了就尴尬了。此时可以用**btracec**预编译一下：

`./btracec HelloWorld.java`

------



接下来，开始一步步讲解脚本的编写。 

### 2. 拦截方法定义

#### 2.1 精准定位

就是HelloWorld的例子，精确定义要监控的类名与方法名。

#### 2.2 正则表达式定位

批量定义需要监控的类与方法。正则表达式需要写在两个 "/" 中间。

下例监控javax.swing下的所有类的所有方法....会非常慢，实际使用时范围还是要窄些。

```java
@OnMethod(clazz="/javax\\.swing\\..*/", method="/.*/")

public static void swingMethods( @ProbeClassName String probeClass, @ProbeMethodName String probeMethod) {
   print("entered " + probeClass + "."  + probeMethod);
}
```

通过在拦截函数的参数定义里注入@ProbeClassName和 @ProbeMethodName参数，告诉脚本实际匹配到的类和方法名。

另一个例子，监控Statement的executeUpdate(), executeQuery() 和 executeBatch() 三个方法：`method = "/execute($|Update|Query|Batch)/" `



#### 2.3 按接口，父类，Annotation定位

要匹配所有Filter类，在接口或父类的名称前面，加个 "+" 就行
`@OnMethod(clazz="+com.vip.demo.Filter", method="doFilter")`

也可以按类或方法上的annotaiton匹配，前面加个@就行
`@OnMethod(clazz="@javax.jws.WebService", method="@javax.jws.WebMethod")`

 

#### 2.4 其他

构造函数的名字是 *"<init>"*
`@OnMethod(clazz="java.net.ServerSocket", method="<init>")`

如果有多个同名的函数，想区分开来，可以在拦截函数上定义不同的参数列表（见4.1）。

 

### 3. 拦截时机

可以为同一个函数的不同Location，分别定义多个拦截函数。

#### 3.1 Kind.Entry与Kind.Return

`@OnMethod(clazz="java.net.ServerSocket", method="bind" )`

不写Location，默认就是刚进入函数的时候(Kind.ENTRY)。

但如果你想获得函数的返回结果或执行时间，则必须把拦截点定在返回时。

```java
OnMethod(clazz = "java.net.ServerSocket", method = "getLocalPort", location = @Location(Kind.RETURN))

public static void onGetPort(@Return int port, @Duration long duration)
```

duration的单位是纳秒，要除以 1,000,000 才是毫秒。



#### 3.2 Kind.Error, Kind.Throw和 Kind.Catch

Throw：异常抛出，Catch：异常被捕获，Error：异常没被捕获而被抛出函数之外，主要用于对某些异常情况的跟踪。

在参数定义里注入一个Throwable的参数，代表异常。

```java
@OnMethod(clazz = "java.net.ServerSocket", method = "bind", location = @Location(Kind.ERROR))

public static void onBind(Throwable exception, @Duration long duration)
```



#### 3.3 Kind.Call与Kind.Line

下例定义监控bind()函数里调用的所有其他函数：

```java
@OnMethod(clazz = "java.net.ServerSocket", method = "bind", location = @Location(value = Kind.CALL, clazz = "/.*/", method = "/.*/", where = Where.AFTER))

public static void onBind(@Self Object self, @TargetInstance Object instance, @TargetMethodOrField String method, @Duration long duration)
```

所调用的类及方法名所注入到@TargetInstance与 @TargetMethodOrField中。

静态函数中，instance的值为空。如果想获得执行时间，必须把Where定义成AFTER。

注意这里，一定不要像下面这样大范围的匹配，否则这性能是神仙也没法救了

`@OnMethod(clazz = "/javax\\.swing\\..*/", method = "/.*/", location = @Location(value = Kind.CALL, clazz = "/.*/", method = "/.*/"))`

下例代码监控是否到达了Socket类的第363行。

```java
@OnMethod(clazz = "java.net.ServerSocket", location = @Location(value = Kind.LINE, line = 363))

public static void onBind4(int line) {
  println("socket bind reach line:363");
}
```

line还可以为-1，然后每行都会打印出来，加参数int line 获得的当前行数。此时会显示函数里完整的执行路径，但肯定又非常慢。



### 4. 打印this，参数 与 返回值

#### 4.1 定义参数注入

```java
import com.sun.btrace.AnyType;

@OnMethod(clazz = "java.io.File", method = "createTempFile", location = @Location(value = Kind.RETURN))
public static void o(@Self Object self, String prefix, String suffix, @Return AnyType result)
```

如果想打印它们，首先按顺序定义用@Self 注释的this， 完整的参数列表，以及用@Return 注释的返回值。

需要打印哪个就定义哪个，不需要的就不要定义。但定义一定要按顺序，比如参数列表不能跑到返回值的后面。

**Self：**

如果是静态函数， self为空。

前面提到，如果上述使用了非JDK的类，命令行里要指定classpath。不过，如前所述，因为BTrace里不允许调用类的方法，所以定义具体类很多时候也没意思，所以self定义为Object就够了。

**参数：**

参数数列表要么不要定义，要定义就要定义完整，否则BTrace无法处理不同参数的同名函数。

如果有些参数你实在不想引入非JDK类，又不会造成同名函数不可区分，可以用AnyType来定义（不能用Object）。

如果拦截点用正则表达式中匹配了多个函数，函数之间的参数个数不一样，你又还是想把参数打印出来时，可以用AnyType[] args来定义。

但不知道是不是当前版本的bug，AnyType[] args 不能和 location＝Kind.RETURN 同用，否则会进入一种奇怪的静默状态，只要有一个函数定义错了，整个BTrace就什么都打印不出来。

**结果**：

同理，结果也可以用AnyType来定义，特别是用正则表达式匹配多个函数的时候，连void都可以表示。





#### 4.2 打印

再次强调，为了保证性能不受影响，BTrace不允许调用任何实例方法。
比如不能调用getter方法（怕在getter里有复杂的计算），只能通过直接反射来读取属性名。


又比如，除了JDK类，其他类toString时只会打印其类名＋System.IdentityHashCode。
println, printArray，都按上面的规律进行，所以只能打打基本类型。

如果想打印一个Object的所有属性，用printFields()来反射。

如果只想反射某个属性，参照下面打印Port属性的写法，注意JDK类与非JDK类的区别：

```java
import java.lang.reflect.Field;

//JDK的类这样写就行
private static Field fdFiled = field("java.io,FileInputStream", "fd");

//非JDK的类，要给出ClassLoader，否则ClassNotFound
private static Field portField = field(classForName("com.vip.demo.MyObject", contextClassLoader()), "port");
public static void onChannelRead(@Self Object self) {
  println("port:" + getInt(portField, self));
}
```



#### 4.3.TLS，拦截函数们之间间的通信机制

如果要多个拦截函数之间要通信，可以使用@TLS定义 ThreadLocal的变量来共享

```java
@TLS
private static int port = -1;

@OnMethod(clazz = "java.net.ServerSocket", method = "<init>")
public static void onServerSocket(int p){
  port = p;
}

@OnMethod(clazz = "java.net.ServerSocket", method = "bind")
public static void onBind(){
  println("server socket at " + port);
}
```



### 5. 典型场景

#### 5.1 打印慢调用

下例打印所有用时超过1毫秒的Filter

```java
@OnMethod(clazz = "+com.vip.demo.Filter", method = "doFilter", location = @Location(Kind.RETURN))
public static void onDoFilter2(@ProbeClassName String pcn,  @Duration long duration) {
  if (duration > 1000000) {
    println(pcn + ",duration:" + (duration / 100000));
  }
}
```

最好能抽取打印耗时的函数，减少代码重复度。

定位到某一个Filter慢了之后，可以直接用Location(Kind.CALL)，进一步找出它里面的哪一步慢了。



#### 5.2 谁调用了这个函数

比如，谁调用了System.gc() ?

```java
@OnMethod(clazz = "java.lang.System", method = "gc")
public static void onSystemGC() {
  println("entered System.gc()");
  jstack();
}
```



#### 5.3 捕捉异常，或进入了某个特定代码行时，this对象及参数的值

按之前的提示，自己组合一下即可。



#### 5.4 打印函数的调用的统计信息

如果你已经看到了这里，那基本也不用我再啰嗦了，自己看Samples里的Histogram.java 和 HistoOnEvent.java

可以用AtomicInteger构造计数器，然后定时(@OnTimer)，或根据事件(@OnEvent)输出结果，ctrl+c后选择发送事件。