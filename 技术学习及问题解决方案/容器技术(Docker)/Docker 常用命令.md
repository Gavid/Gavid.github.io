## 常用docker命令

```bash
# 登录/登出
# 线上harbar: http://harboroffline.xxv5.cn/
# 线下harbar: http://harbor.xxcorp.com/
docker logout http://harbor.xxcorp.com/
docker login http://harbor.xxcorp.com/

docker images | grep xxx # 查看镜像
docker pull xxx # 拉取xxx最新镜像
docker tag xxx yyy # 将xxx打tag ,命名为yyy
docker push yyy # 将yyy推送到远程仓库

docker ps [OPTIONS]
  OPTIONS说明：
    -a :显示所有的容器，包括未运行的。
    -f :根据条件过滤显示的内容。
    --format :指定返回值的模板文件。
    -l :显示最近创建的容器。
    -n :列出最近创建的n个容器。
    --no-trunc :不截断输出。
    -q :静默模式，只显示容器编号。
    -s :显示总的文件大小。
  输出详情介绍：
    CONTAINER ID: 容器 ID。
    IMAGE: 使用的镜像。
    COMMAND: 启动容器时运行的命令。
    CREATED: 容器的创建时间。
    STATUS: 容器状态。
      状态有7种：created（已创建）、restarting（重启中）、running（运行中）、removing（迁移中）、paused（暂停）、exited（停止）、dead（死亡）
    PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
    NAMES: 自动分配的容器名称。
    
# 创建容器
# 并直接进入docker容器中
docker run -it --name {容器名称} {镜像ID}:{版本号} /bin/bash
# 返回容器id 【需要start后才能进入容器中】
docker create {镜像ID}:{版本号}

# start, 容器从 Exited --> Running
docker start [docker_name|docker_id]
# stop, 容器从 Running --> Exited
docker stop [docker_name|docker_id]
# 进入容器进行操作
docker exec -it docker_name /bin/bash
# 为容器重命名
docker rename docker_name new_docker_name
# 删除容器
docker rm 容器id
# 删除镜像
docker rmi 镜像ID

```

## 