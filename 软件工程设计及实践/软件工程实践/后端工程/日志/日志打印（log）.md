# 日志打印（log）

1. Pom.xml

   ```xml
   <dependency>
     <groupId>org.apache.logging.log4j</groupId>
     <artifactId>log4j-core</artifactId>
     <version>2.14.1</version>
   </dependency>
   <dependency>
     <groupId>org.slf4j</groupId>
     <artifactId>log4j-over-slf4j</artifactId>
     <version>1.7.26</version>
   </dependency>
   <dependency>
     <groupId>org.apache.logging.log4j</groupId>
     <artifactId>log4j-slf4j-impl</artifactId>
     <version>2.14.1</version>
     <exclusions>
       <exclusion>
         <artifactId>log4j-api</artifactId>
         <groupId>org.apache.logging.log4j</groupId>
       </exclusion>
       <exclusion>
         <artifactId>log4j-core</artifactId>
         <groupId>org.apache.logging.log4j</groupId>
       </exclusion>
     </exclusions>
   </dependency>
   <dependency>
     <groupId>org.apache.logging.log4j</groupId>
     <artifactId>log4j-jcl</artifactId>
     <version>2.14.1</version>
     <exclusions>
       <exclusion>
         <artifactId>log4j-api</artifactId>
         <groupId>org.apache.logging.log4j</groupId>
       </exclusion>
     </exclusions>
   </dependency>
   ```

2. log4j2.xml （该文件放到resource的根目录下，使用了Spring相关框架，可将文件名修改为`log4j2-spring.xml`）

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <Configuration status="${env:LOG_LEVEL:-warn}">
   
       <Properties>
           <Property name="layout">%d{yyyy-MM-dd HH:mm:ss.SSS}{GMT+8} %-5p [%t] %c{2.} - %m%n%ex</Property>
       </Properties>
   
       <Appenders>
           <Console name="console" target="SYSTEM_OUT">
               <PatternLayout pattern="${layout}"/>
           </Console>
           <Socket name="udp1" host="${sys:udpLogHost:-10.148.14.94}" port="${sys:udpLogPort:-15010}" protocol="UDP">
               <PatternLayout pattern="%m"/>
           </Socket>
   
           <Socket name="udp2" host="${sys:udpLogHost:-10.148.14.71}" port="${sys:udpLogPort:-15010}" protocol="UDP">
               <PatternLayout pattern="%m"/>
           </Socket>
       </Appenders>
   
       <Loggers>
   <!--        <Logger name="org.springframework" level="${env:LOG_LEVEL:-debug}"/>-->
           <Logger name="wcs-udp-log" level="info" additivity="false">
               <AppenderRef ref="udp1"/>
   <!--            <AppenderRef ref="udp2"/>-->
               <AppenderRef ref="console"/>
           </Logger>
           <Root level="${env:ROOT_LOG_LEVEL:-error}">
               <AppenderRef ref="console"/>
           </Root>
       </Loggers>
   
   </Configuration>
   ```

   
