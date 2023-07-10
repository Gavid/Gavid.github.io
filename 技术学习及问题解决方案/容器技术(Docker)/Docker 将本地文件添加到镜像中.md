## docker将本地文件添加到镜像中

有两种实现方案

1. docker cp命令行(docker cp只能拷贝到容器中，不能直接拷贝到镜像)

  ```bash
# 1.1 启动一个容器
docker run -it nginx /bin/bash
# 然后ctrl+d退出，利用docker ps -a可以看到容器的ID

# 1.2 拷贝文件: 拷贝一张图片到容器的workspace目录下
# 从宿主机拷文件到容器里面
#	docker cp 本地路径 容器名或者id:容器存储路径
# 从容器拷文件到宿主机里面
# docker cp 容器名或者id:容器路径 本地路径
docker cp /home/img_666.jpg 6267:/workspace

# 1.3 将容器生成镜像
docker commit -m 'test docker cp' -a 'chenjun_csu@163.com' 6269 nginx:v2

  ```

2. dockerfile

  dockerfile的内容为:

  ```dockerfile
FROM nginx                    # 利用docker hub中nginx镜像

ADD ./workspace/img_666.jpg workspace/ 			# 添加一张图片
WORKDIR /workspace								# 设定工作目录
  ```

  进入镜像进行验证。