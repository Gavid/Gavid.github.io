# Antlr4 技术调研

1. [项目代码地址](https://github.com/antlr/antlr4)

2. [简要教程](https://wizardforcel.gitbooks.io/antlr4-short-course/content/installing-antlr.html)

3. 操作记录

   ```bash
   ##### 简单文法
   grammar Hello;               // 定义文法的名字
   
   s  : 'hello' ID ;            // 匹配关键字hello和标志符
   ID : [a-z]+ ;                // 标志符由小写字母组成
   WS : [ \t\r\n]+ -> skip ;    // 跳过空格、制表符、回车符和换行符
   #####
   
   # 1.生成识别器, vi antlr ; chmod u+x antlr ; ./antlr hello.g4
   #!/bin/sh
   java -cp antlr-4.5.3-complete.jar org.antlr.v4.Tool $*
   
   ls Hello*
   # Hello.tokens  HelloLexer.interp  HelloLexer.java   HelloBaseListener.java  Hello.g
   # Hello.interp  HelloLexer.tokens  HelloParser.java  HelloListener.java
   
   # 2.编译代码, vi compile ; chmod u+x compile ; ./compile Hello*.java
   #!/bin/sh
   javac -cp antlr-4.5.3-complete.jar $*
   
   # 3.测试文法，vi grun ; chmod u+x grun ; 
   #!/bin/sh
   java -cp antlr-4.5.3-complete.jar org.antlr.v4.gui.TestRig $*
   #  以记号列表形式展示
   ./grun Hello s -tokens  # Hello是文法的名字。s是开始的规则名字。
   # 以LISP风格的文本形式查看记号：
   ./grun Hello s -tree
   # 以语法分析树形式展示
   ./grun Hello s -gui
   # hello world  # 输入并按回车
   # EOF          # Linux系统输入Ctrl+D或Windows系统输入Ctrl+Z并按回车
   
   # TestRig可用的所有参数：
   -tokens 打印出记号流。
   -tree 以LISP风格的文本形式打印出语法分析树。
   -gui 在对话框中可视化地显示语法分析树。
   -ps file.ps 在PostScript中生成一个可视化的语法分析树表示，并把它存储在file.ps文件中。
   -encoding encodingname 指定输入文件的编码。
   -trace 在进入/退出规则前打印规则名字和当前的记号。
   -diagnostics 分析时打开诊断消息。此生成消息仅用于异常情况，如二义性输入短语。
   -SLL 使用更快但稍弱的分析策略。
   
   # 到此将输出记号列表
   # @1表示记号索引；6:10表示记号开始与结束的位置；<2>表示记号类型，具体数值和类型存储在后缀名为tokens的文件中；最后的1:6表示记号在第一行，从第6个字符开始
   [@0,0:4='hello',<1>,1:0]
   [@1,6:10='world',<2>,1:6]
   [@2,13:12='<EOF>',<-1>,2:0]
   
   
   
   ```

![img](./img/Antlr4 技术调研/basic-data-flow.png)

识别器代码生成命令

```bash
java -cp antlr-4.5.3-complete.jar org.antlr.v4.Tool $* -package com.bj58.search.wcs.client.script.antlr -Dlanguage=Java -listener -encoding utf-8 -visitor -lib /Users/baijiahao/IdeaProjects/wcs-agent/src/main/resources/antlr /Users/baijiahao/IdeaProjects/wcs-agent/src/main/resources/antlr/PainlessParser.g4 /Users/baijiahao/IdeaProjects/wcs-agent/src/main/resources/antlr/PainlessLexer.g4
```



# 