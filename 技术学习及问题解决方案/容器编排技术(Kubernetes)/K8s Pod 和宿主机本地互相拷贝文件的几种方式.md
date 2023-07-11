### 概要描述

------

该文档总结了 K8s Pod 中和本地宿主机文件传输的几种方式，有需要的时候可供参考。欢迎指正和添加。下文提到的多种方法中，选择一种适合自己的就行。
本文档的案例均为在 Centos 7 上实现，命令均通过 root 用户执行。

### 详细说明

------

**1. kubectl cp**
 **使用前提**：Pod 中对应的 container 中安装了 tar 命令，该方式不用进入到 Pod 内
 **注意**：有的TOS 里该命令貌似是不能用的

- 首先列出 Pod，获取 Pod 的 Name, mycentos 为 pod name 中的关键字

```bash
# kubectl get pods -o wide | grep -i mycentos

NAME                        READY   STATUS    RESTARTS   AGE
mycentos-7b59b5b755-8rbgc   1/1     Running   0          5h20m
mycentos-7b59b5b755-vm6ld   1/1     Running   1          5h31m
```

- 宿主机 –> Pod

  将宿主机 /tmp/test_pod.txt 拷贝到 mycentos-7b59b5b755-8rbgc 对应的第一个容器中的 /root 目录下，拷贝目录也是一样的。default 为 namespace

```bash
# kubectl cp /tmp/test_pod.txt default/mycentos-7b59b5b755-8rbgc:/root
```

- Pod –> 宿主机

  将 mycentos-7b59b5b755-8rbgc 中 /root/from_pod.txt 文件拷贝到宿主机 /tmp 目录下，并命名为 from_pod.new

```bash
# kubectl cp default/mycentos-7b59b5b755-8rbgc:/root/from_pod.txt  /tmp/from_pod.new
```

  将 mycentos-7b59b5b755-8rbgc 中 /root/pod_dir 目录下的文件拷贝到宿主机 /tmp 目录下

```
# kubectl cp default/mycentos-7b59b5b755-8rbgc:/root/pod_dir /tmp
```

 **Tips**：
  Pod –> 宿主机时，拷贝文件需要指定目标文件名；拷贝目录可以不指定



**2. scp**
 **使用前提**：宿主机和 Pod 的 container 中都安装了 scp 命令，需要宿主机的密码，该方式需要进入到 Pod 内
 执行如下命令进入 pod，mycentos-7b59b5b755-8rbgc 替换为相应 pod name。

```bash
# kubectl exec -it  mycentos-7b59b5b755-8rbgc /bin/bash
```

- 宿主机 –> Pod

  将宿主机 /tmp/test_pod.txt 拷贝到 mycentos-7b59b5b755-8rbgc 对应的第一个容器中 /root 目录下,宿主机 ip 为 172.22.202.69，用户 root

```bash
# scp -rv root@172.22.202.69:/tmp/test_pod.txt /root 
```

- Pod –> 宿主机

  将 mycentos-7b59b5b755-8rbgc 中 /root/from_pod.txt 文件拷贝到宿主机 /tmp 目录下，宿主机 ip 为 172.22.202.69，用户 root

```bash
# scp -rv /root/from_pod.txt root@172.22.202.69:/tmp
```

 **Tips**: 安装 scp 命令 `yum install -y openssh-clients`



**3. docker cp**
  获取 pod 名为 mycentos-7b59b5b755-8rbgc 对应 container 的 id

```bash
# kubectl describe pod/mycentos-7b59b5b755-8rbgc | grep -iE "\s.*container id.*|Node:\s*.*" -B 2 -A 2

Namespace:      default
Priority:       0
Node:           172.22.202.70/172.22.202.70
Start Time:     Wed, 07 Aug 2019 12:58:17 +0800
Labels:         pod-template-hash=7b59b5b755
--
Containers:
  mycentos:
    Container ID:  docker://5c73edb95237ab7e4af2d85aad9674899492134e8addc20dfc05e2a598acd7eb
    Image:         centos
    Image ID:      docker-pullable://centos@sha256:a799dd8a2ded4a83484bbae769d97655392b3f86533ceb7dd96bbac929809f3c
```

  从上面的结果可以得知该 pod 对应的 container 运行在 172.22.202.70 节点上，相应的 container id 为：5c73edb95237ab7e4af2d85aad9674899492134e8addc20dfc05e2a598acd7eb

- 宿主机 –> Pod

  到 container 运行的 node 172.22.202.70 上，将 /tmp/test_pod.txt 拷贝到容器里 /root 目录下

```bash
# docker cp /tmp/test_pod.txt 5c73edb95237ab7e4af2d85aad9674899492134e8addc20dfc05e2a598acd7eb:/root
```

- Pod –> 宿主机

  到 container 运行的 node 172.22.202.70 上，将容器内 /root/from_pod.txt 拷贝到本地 /tmp 目录下

```bash
# docker cp 5c73edb95237ab7e4af2d85aad9674899492134e8addc20dfc05e2a598acd7eb:/root/from_pod.txt /tmp
```



**4. lftp**
 **使用前提**：Pod 内容器安装了 lftp 工具，本地宿主机安装并启动了 vsftpd，该方式需要进入到 Pod 内
 执行如下命令进入 pod，mycentos-7b59b5b755-8rbgc 替换为相应 pod name。

```bash
# kubectl exec -it  mycentos-7b59b5b755-8rbgc /bin/bash
```

 lftp 连接到 172.22.202.69 的 21 端口，用 root 用户，输入密码后即可进行文件传输

```bash
# lftp 172.22.202.69 -p 21 -u root
```

- 宿主机 –> Pod

 lftp 内命令传输示例，将本地 /tmp/pod_dir 目录下的所有文件传输到 Pod 的 /root 目录下

```bash
# mget /tmp/pod_dir/* -O /root 
```

- Pod –> 宿主机

 将 Pod 的 /root 目录下的 from_pod.txt 文件传输到本地 /tmp 目录下

```bash
# mput /root/from_pod.txt -O /tmp 
```

 **Tips**:
  **vsftpd 的安装**：` yum install -y vsftpd && sleep 2 && systemctl start vsftpd && sleep 2 && systemctl enable vsftpd && systemctl status vsftpd`
  **lftp 的安装**：` yum install -y lftp`



**5. cp**
 参考第 3. 部分先获取 Pod 对应的 container id，以下以容器 5c73edb95237ab7e4af2d85aad9674899492134e8addc20dfc05e2a598acd7eb 举例。
 找到 5c73edb95237ab7e4af2d85aad9674899492134e8addc20dfc05e2a598acd7eb 容器在本地的读写层目录

```bash
# docker inspect -f '{{.GraphDriver.Data.UpperDir}}' 5c73edb95237ab7e4af2d85aad9674899492134e8addc20dfc05e2a598acd7eb
```

 上述执行结果为：/var/lib/docker/overlay2/e57f783cce10c2d14ea48b6194f3796f361fa9e045e633038b604c32ad9fce1a/diff

```bash
# ls -lrt /var/lib/docker/overlay2/e57f783cce10c2d14ea48b6194f3796f361fa9e045e633038b604c32ad9fce1a/diff

total 8
drwxr-xr-x 8 root root   75 Mar  6 01:34 usr
drwxr-xr-x 6 root root   48 Mar  6 01:34 var
drwxr-xr-x 2 root root   28 Aug  7 20:43 dev
drwxr-xr-x 4 root root 4096 Aug  7 21:05 etc
drwxrwxrwt 2 root root    6 Aug  7 21:05 tmp
drwxr-xr-x 3 root root   20 Aug  7 21:05 run
dr-xr-x--- 5 root root 4096 Aug  7 22:40 root
```

- 宿主机 –> Pod

  直接在本地操作，将 /tmp/test_pod.txt 文件拷贝到 Pod 的 /root 目录，然后在 Pod 的 /root 目录下便可看到该文件

```bash
# cp /tmp/test_pod.txt /var/lib/docker/overlay2/e57f783cce10c2d14ea48b6194f3796f361fa9e045e633038b604c32ad9fce1a/diff/root/
```



 **总结**：其实 Pod 也可以当作是另一个服务器，所以 Linux 中服务器间传递文件的一些工具如 ftp 等，理论上来说都可以用来实现本地主机和 Pod 内部文件传输。