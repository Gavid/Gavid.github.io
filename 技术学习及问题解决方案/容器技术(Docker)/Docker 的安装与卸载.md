## Docker 的安装与卸载

```bash
# https://www.runoob.com/docker/centos-docker-install.html
# 安装命令
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun  # 或者 curl -sSL https://get.daocloud.io/docker | sh

# 卸载命令
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine

# 删除安装包
yum remove docker-ce
# 删除镜像、容器、配置文件等内容：
rm -rf /var/lib/docker
```

