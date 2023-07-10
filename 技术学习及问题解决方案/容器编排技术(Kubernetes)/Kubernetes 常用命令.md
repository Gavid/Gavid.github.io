## Kubernetes 常用命令

### 查看节点信息

  ```bash
kubectl get nodes
kubectl get nodes -o wide
nodes ：节点
O wide :表示打印更多的信息出来
  ```

### 设置节点状态为：可调度 / 不可调度

```bash
# 设置节点不可调度
kubectl cordon [节点名称]
# 设置节点可调度
kubectl uncordon [节点名称]
```

### 查看命名空间

  ```bash
kubectl get ns
ns ：命名空间
  ```

### 查看默认命名空间的pods

  ```bash
kubectl get pods -o wide
打印pods的信息
  ```

### 查看指定名称空间的pods

  ```bash
kubectl get pods -n kube-system
指定打印命名空间为kube-system的pods有哪些
  ```

### 查看所有名称空间的pods

  ```bash
kubectl get pods --all-namespaces
打印所有命名空间下的pods
  ```

### 监控pod进度

  ```bash
watch kubectl get pod -n kube-system -o wide
监控命名空间kube-system下的pod进度
  ```

### 部署一个tomcat

  ```bash
kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8
部署一个tomcat:6.0.53-jre8类型的镜像，名称为：tomcat6
  ```

### 暴露nginx访问

  ```bash
kubectl expose deployment tomcat6 --port=80 --target-port=8080 --type=NodePort 
在master上执行
--type=NodePort ：为service随机分配端口映射到pod的80，pod的80再映射到容器tomcat的8080
  ```

### 查看服务

  ```bash
kubectl get svc -o wide
svc：查看服务，为缩写
  ```

### 查看所有的资源：

  ```bash
kubectl get all
kubectl get all -o wide
查看所有资源情况
  ```

### 查看部署

```bash
kubectl get deployment
查看所有部署情况
```

### 查看pod运行日志

```bash
kubectl describe pods -n ingress-nginx  nginx-ingress-controller-bdhw2
查看命名空间ingress-nginx下的pod名为nginx-ingress-controller-bdhw2信息
```

### 查看pod容器日志

```bash
kubectl -n wcs-online logs -f {podname}
```

### 缩放服务

```bash
kubectl scale --replicas=3 deployment tomcat6
把部署扩容到3份
kubectl scale -n wcs-online deployment sink-query-proxy-meishidoc4-v5.1.7-meishi-doc --replicas=0
```

### 删除部署

```bash
kubectl delete deployment.apps/tomcat6
删除deployment部署信息会自动删除replicaset和pod，只留下service
```

### 删除服务

```bash
kubectl delete service/tomcat6
删除service/tomcat6这个服务
```

### 只生成部署文件yml

```bash
kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8 --dry-run -o yaml > tomcat6.yml
创建一个镜像为tomcat:6.0.53-jre8的部署文件
```

### 只生成暴露文件（service文件）yml

```bash
kubectl expose deployment tomcat6 --port=80 --target-port=8080 --type=NodePort --dry-run -o yaml > tomcat6--expose.yml
使用expose创建一个暴露的service。
```

### 查看某个pod的具体定义信息以yaml格式输出

```bash
kubectl get pods tomcat6-5f7ccf4cb9-gkqjr -o yaml
输出某个pods为yaml文件
```

### 应用部署文件

```bash
kubectl apply -f tomcat6.yaml
运行该文件
```

### 查看某个pod中某个容器

```bash
kubectl -n wcs-online exec -it {pod name} bash -c {容器 name}
```

