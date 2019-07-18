@[TOC](slf4 + logback日志组件的介绍，以及在javaWeb工程中的详细应用)
# 1. slf4 与 logback 的简单介绍
* slf4j： slf4j是一个日志系统的封装，对外提供统一的API，不提供日志具体实现。
* logback：和log4j同为日志的一种具体实现。（Logback是由log4j创始人设计的又一个开源日志组件）
# 2. logback相对于log4j的优点
 * 1、**更快的实现**  Logback的内核重写了，在一些关键执行路径上性能提升10倍以上。而且logback不仅性能提升了，初始化内存加载也更小了。
 * 2、**非常充分的测试**  Logback经过了几年，数不清小时的测试。Logback的测试完全不同级别的。在作者的观点，这是简单重要的原因选择logback而不是log4j。
* 3、**Logback-classic非常自然实现了SLF4j**    Logback-classic实现了 SLF4j。在使用SLF4j中，你都感觉不到logback-classic。而且因为logback-classic非常自然地实现了SLF4J，  所 以切换到log4j或者其他，非常容易，只需要提供成另一个jar包就OK，根本不需要去动那些通过SLF4JAPI实现的代码。
* 4、**非常充分的文档**  官方网站有两百多页的文档。
* 5、**自动重新加载配置文件**  当配置文件修改了，Logback-classic能自动重新加载配置文件。扫描过程快且安全，它并不需要另外创建一个扫描线程。这个技术充分保证了应用程序能跑得很欢在JEE环境里面。
* 6、**Lilith**   Lilith是log事件的观察者，和log4j的chainsaw类似。而lilith还能处理大数量的log数据 。
* 7、**谨慎的模式和非常友好的恢复**  在谨慎模式下，多个FileAppender实例跑在多个JVM下，能 够安全地写道同一个日志文件。RollingFileAppender会有些限制。Logback的FileAppender和它的子类包括 RollingFileAppender能够非常友好地从I/O异常中恢复。
* 8、**配置文件可以处理不同的情况**   开发人员经常需要判断不同的Logback配置文件在不同的环境下（开发，测试，生产）。而这些配置文件仅仅只有一些很小的不同，可以通过,和来实现，这样一个配置文件就可以适应多个环境。
* 9、**Filters（过滤器）**  有些时候，需要诊断一个问题，需要打出日志。在log4j，只有降低日志级别，不过这样会打出大量的日志，会影响应用性能。在Logback，你可以继续 保持那个日志级别而除掉某种特殊情况，如alice这个用户登录，她的日志将打在DEBUG级别而其他用户可以继续打在WARN级别。要实现这个功能只需 加4行XML配置。可以参考MDCFIlter 。
* 10、**SiftingAppender（一个非常多功能的Appender）**  它可以用来分割日志文件根据任何一个给定的运行参数。如，SiftingAppender能够区别日志事件跟进用户的Session，然后每个用户会有一个日志文件。
* 11、**自动压缩已经打出来的log**  RollingFileAppender在产生新文件的时候，会自动压缩已经打出来的日志文件。压缩是个异步过程，所以甚至对于大的日志文件，在压缩过程中应用不会受任何影响。
* 12、**堆栈树带有包版本**  Logback在打出堆栈树日志时，会带上包的数据。
* 13、**自动去除旧的日志文件**  通过设置TimeBasedRollingPolicy或者SizeAndTimeBasedFNATP的maxHistory属性，你可以控制已经产生日志文件的最大数量。如果设置maxHistory 12，那那些log文件超过12个月的都会被自动移除。
# 3. 应该在什么地方写日志
* 当你遇到问题的时候，只能通过debug功能来确定问题，你应该考虑打日志，良好的系统，是可以通过日志进行问题定位的；
* 当你碰到if…else 或者 switch这样的分支时，要在分支的首行打印日志，用来确定进入了哪个分支；
* 经常以功能为核心进行开发，你应该在提交代码前，可以确定通过日志可以看到整个流程；
# 4. 在java项目中的具体应用
## 4.1 基于maven的slf4j+logback pom.xml配置
```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.10</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.2</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.1.2</version>
</dependency>
```
## 4.2 配置logback.xml文件对日志组件进行初始化
### 4.2.1 默认配置步骤
 (1). 尝试在 classpath 下查找文件 logback-test.xml；

 (2). 如果文件不存在，则查找文件 logback.xml；

 (3). 如果两个文件都不存在，logback 用 Bas icConfigurator 自动对自己进行配置，这会导致记录输出到控制台。
 ### 4.2.2 自定义配置步骤
 * 在java项目工程目录中找到“ src --> main --> resources”目录，并在此目录下新建 logback.xml 文件。
 * logback.xml文件的模板及详细注释如下：
	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<!--
	scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
	scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒当scan为true时，此属性生效。默认的时间间隔为1分钟。
	debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
	-->
	<configuration scan="false" scanPeriod="60 seconds" debug="false">
	    <!-- 定义日志的根目录 -->
	    <property name="LOG_HOME" value="/app/log" />
	    <!-- 定义日志文件名称 -->
	    <property name="appName" value="netty"></property>
	    <!-- ch.qos.logback.core.ConsoleAppender 表示控制台输出 -->
	    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
	        <Encoding>UTF-8</Encoding>
	        <!--
	        日志输出格式：%d表示日期时间，%thread表示线程名，%-5level：级别从左显示5个字符宽度
	        %logger{50} 表示logger名字最长50个字符，否则按照句点分割。 %msg：日志消息，%n是换行符
	        -->
	        <layout class="ch.qos.logback.classic.PatternLayout">
	            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
	        </layout>
	    </appender>
	    
	    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->  
	    <appender name="appLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
	        <Encoding>UTF-8</Encoding>
	        <!-- 指定日志文件的名称 -->  
	        <file>${LOG_HOME}/${appName}.log</file>
	        <!--
	        当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名
	        TimeBasedRollingPolicy： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。
	        -->
	        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
	            <!--
	            滚动时产生的文件的存放位置及文件名称 %d{yyyy-MM-dd}：按天进行日志滚动 
	            %i：当文件大小超过maxFileSize时，按照i进行文件滚动
	            -->
	            <fileNamePattern>${LOG_HOME}/${appName}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
	            <!-- 
	            可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每天滚动，
	            且maxHistory是365，则只保存最近365天的文件，删除之前的旧文件。注意，删除旧文件是，
	            那些为了归档而创建的目录也会被删除。
	            -->
	            <MaxHistory>365</MaxHistory>
	            <!-- 
	            当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动 注意此处配置SizeBasedTriggeringPolicy是无法实现按文件大小进行滚动的，必须配置timeBasedFileNamingAndTriggeringPolicy
	            -->
	            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
	                <maxFileSize>100MB</maxFileSize>
	            </timeBasedFileNamingAndTriggeringPolicy>
	        </rollingPolicy>
	        <!--
	        日志输出格式：%d表示日期时间，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %logger{50} 表示logger名字最长50个字符，否则按照句点分割。 %msg：日志消息，%n是换行符
	        -->     
	        <layout class="ch.qos.logback.classic.PatternLayout">
	            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n</pattern>
	        </layout>
	    </appender>
	
	    <!-- 
	    logger主要用于存放日志对象，也可以定义日志类型、级别
	    name：表示匹配的logger类型前缀，也就是包的前半部分
	    level：要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR
	    additivity：作用在于children-logger是否使用 rootLogger配置的appender进行输出，false：表示只用当前logger的appender-ref，true：表示当前logger的appender-ref和rootLogger的appender-ref都有效
	    -->
	    <!-- hibernate logger -->
	    <logger name="org.hibernate" level="error" />
	    <!-- Spring framework logger -->
	    <logger name="org.springframework" level="error" additivity="false"></logger>
	
	    <logger name="com.creditease" level="info" additivity="true">
	        <appender-ref ref="appLogAppender" />
	    </logger>
	
	    <!-- 
	    root与logger是父子关系，没有特别定义则默认为root，任何一个类只会和一个logger对应，
	    要么是定义的logger，要么是root，判断的关键在于找到这个logger，然后判断这个logger的appender和level。 
	    -->
	    <root level="info">
	        <appender-ref ref="stdout" />
	        <appender-ref ref="appLogAppender" />
	    </root>
	</configuration> 
	```
## 4.3 在java工程中如何使用Logback
1. 首先导入依赖包
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
```
2. 初始化Logger对象
```java
// 或者 Logger logger = LoggerFactory.getLogger(getClass());
static final Logger logger = LoggerFactory.getLogger(MyClassName.class);
```
3. 执行log操作
```java
logger.debug("debug");
logger.info("info");
```
参考文献：
https://mp.weixin.qq.com/s/37VOP-iWrWgvbftt5UPj0g
https://blog.csdn.net/zgmzyr/article/details/8267072
https://www.jianshu.com/p/09b9b50f32ef
