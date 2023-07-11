# **[Greys入门](https://github.com/oldmanpushcart/greys-anatomy/wiki/Getting-Started)**

## 启动Greys

### 参数说明

```
./greys <PID>[@IP:PORT]
```

- **PID：**目标Java进程ID（请确保执行当前执行命令的用户必须有足够的权限操作对应的Java进程）
- **IP：**目标服务器IP地址，当远程服务开启之后，其他人可以通过指定IP的形式加载到对应目标机器的Java进程中，从而实现远程协助。专门用于解决目标主机账号没有权限，但对方兄弟却非常需要你支援的时候。Greys允许多个用户同时访问，并且各自的命令不会相互干扰执行。
- **PORT：**目标服务器端口号，设计端口号的初心则是希望解决同台机器上存在多个Java进程需要被Greys分析的情况，默认的端口号是3658，如果不做区分则会引起端口冲突。

### 启动范例

- 如果不指定**IP**和**PORT**，默认是**127.0.0.1**和**3658**

  ```
  ./greys.sh 12345
  ```

  等价于

  ```
  ./greys.sh 12345@127.0.0.1
  ```

  等价于

  ```
  ./greys.sh 12356@127.0.0.1:3658
  ```

### 会话与任务

Greys是一个C/S架构的程序，所以当Client访问到Server时，Server会维护一个session（会话），以及session的心跳、超时机制。事务（Tx）机制则是建立在session的基础上，所有的命令交互都会创建一个事务，并且产生对应的队列进行输出缓冲。

事务伴随着命令的生命周期而存在，命令分两种：

- 立即返回

  立即返回的命令定义是：敲下命令后Server端立即返回最终结果，后续无持续反馈信息，释放Client对输入的锁定，重新开放让用户输入信息，比如**version**、**sc**、**sm**等。

- 等待中止

  等待中止的命令则是需要用户主动输入**Ctrl+D**完成的命令中止操作。命令执行后无法立即返回最终结果，而是不断的将中间产生的输出源源不断的输出到客户端中，这种命令比如**stack**、**monitor**等。

当session关闭时，所有挂在session的事务也会立即被关闭。

## 重要命令

| 命令                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [help](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#help) | 查看命令的帮助文档，每个命令和参数都有很详细的说明           |
| [sc](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#sc) | 查看JVM已加载的类信息                                        |
| [sm](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#sm) | 查看已加载的方法信息                                         |
| [monitor](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#monitor) | 方法执行监控                                                 |
| [trace](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#trace) | 渲染方法内部调用路径，并输出方法路径上的每个节点上耗时       |
| [ptrace](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#ptrace) | 强化版的`trace`命令。通过指定渲染路径，并可记录下路径中所有方法的入参、返值；与`tt`命令联动。 |
| [watch](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#watch) | 方法执行数据观测                                             |
| [tt](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#tt) [详细用法](https://github.com/oldmanpushcart/greys-anatomy/wiki/TimeTunnel) | 方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测 |
| [stack](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#stack) | 输出当前方法被调用的调用路径                                 |
| [version](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#version) | 输出当前目标Java进程所加载的Greys版本号                      |
| [quit](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#quit) | 退出greys客户端                                              |
| [shutdown](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#shutdown) | 关闭greys服务端                                              |
| [reset](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#reset) | 重置增强类，将被greys增强过的类全部还原                      |
| [jvm](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#jvm) | 查看当前JVM的信息                                            |

## 重要类结构

用于watch/tt命令，grovvy表达式的重要数据结构

```
Advice
   |
   +--ClassLoader loader    (ClassLoader)
   +--Class<?>    clazz     (目标类)
   +--GaMethod    method    (目标方法，包括普通方法和构造函数)
   +--Object[]    params    (调用参数)
   +--Object      returnObj (返回值)
   `--Throwable   throwExp  (抛出异常)
```

在表达式中直接使用变量名，例如

```
watch -b *StringUtil* isEmpty params[0].length()
```