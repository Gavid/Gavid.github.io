## docker使用基础镜像打镜像

1. 创建容器

   ```bash
   # 拉取镜像
   docker pull {centos:7}
   
   # 创建容器
   docker run -id --name {centos7} {centos:7}
   ```

2. 拷贝资源

   ```bash
   docker cp xxx.tar.gz {centos7}:/root
   ···
   ```

3. 进入容器进行检查或执行一些操作

   ```bash
   # 进入容器
   docker exec -it centos7 /bin/bash
   # 切换到 /root 目录
   cd root/
   ···
   ```

4. 构建镜像

   ```bash
   docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
   # -a：提交的镜像作者；
   # -c：使用 Dockerfile 指令来创建镜像；
   # -m：提交时的说明文字；
   # -p：在 commit 时，将容器暂停；
   docker commit -a="xiaoyang" -m="jdk11 and tomcat" {centos7} {mycentos:7}
   ```

5. 记得上传镜像到仓库中

   ```bash
   docker push {mycentos:7}
   ```

## docker使用Dockerfile打镜像

1. 创建镜像所在文件夹及Dockerfile文件

   ```bash
   mkdir docker
   cd docker
   touch Dockerfile
   ```

2. 编辑Dockerfile

   ```dockerfile
   # This is a comment
   FROM ubuntu:18.04
   RUN apt-get update && apt-get install -y ruby ruby-dev vim
   RUN gem install docker
    
   #切换工作目录，进入docker后，会自动进入/home目录
   WORKDIR /home
    
   # 创建docker环境的相对文件夹
   RUN mkdir -p /home/lib
   RUN mkdir -p /home/SDK
   RUN mkdir -p /home/code
   RUN pwd
    
   #建立 fw用户，并切换至fw用户
   RUN groupadd -r fw && useradd -r -g fw fw
   USER fw
   ```

   格式说明：

     每行命令都是以INSTRUCTION statement形式，就是命令+清单的模式。命令要大写，“#”是注解。

   | 命令                               | 作用                                                         |
   | ---------------------------------- | ------------------------------------------------------------ |
   | FROM image_name:tag                | 定义了使用哪个基础镜像启动构建流程                           |
   | MAINTAINER user_name               | 声明镜像的创建者                                             |
   | ENV key value                      | 设置环境变量 (可以写多条)                                    |
   | RUN command                        | 是Dockerfile的核心部分(可以写多条)                           |
   | ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
   | COPY source_dir/file dest_dir/file | 和ADD相似，但是如果有压缩文件并不能解压                      |
   | WORKDIR path_dir                   | 设置工作目录                                                 |
   | EXPOSE port1 prot2                 | 用来指定端口，使容器内的应用可以通过端口和外界交互           |
   | CMD argument                       | 在构建容器时使用，会被docker run 后的argument覆盖            |
   | ENTRYPOINT argument                | 和CMD相似，但是并不会被docker run指定的参数覆盖              |
   | VOLUME                             | 将本地文件夹或者其他容器的文件挂载到容器中                   |

3. 创建镜像

   命令： `docker build -t gcs/docker_img:v1 .`

   > docker build 是docker创建镜像的命令
   > -t 是标识新建的镜像属于 gcs的
   > docker_img是仓库的名称
   > ：v1 是tag
   > “.”是用来指明我们使用的是当前目录的Dockerfile文件

## 制作镜像的时候报错解决方案

1. devmapper: Thin Pool has 162394 free data blocks which is less than minimum required 163840 free data blocks. Create more free space in thin pool or use dm.min_free_space option to change behavior

   解决方案：

   > 运行下面三个命令： 
   > // 注意，逐次运行，逐次制作镜像，直到制作镜像成功。以下三个命令执行时可能出错是正常的。
   >
   > 清理exited进程：
   >
   > ```bash
   > docker rm $(docker ps -q -f status=exited)
   > ```
   >
   > 清理dangling volumes：
   >
   > ```bash
   > docker volume rm $(docker volume ls -qf dangling=true) 
   > ```
   >
   > 清理dangling image：
   >
   > ```bash
   > docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
   > ```

