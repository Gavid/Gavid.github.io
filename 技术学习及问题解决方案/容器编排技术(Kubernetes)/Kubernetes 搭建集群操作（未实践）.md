**搭建kubernetes集群操作指南**

**目录**

**环境介绍**

域名： 方案deprecated

wcs.k8sv2-etcd.58dns.org 访问etcd 域名

wcs.k8sv2-apiserver.58dns.org 访问kube-apiserver 域名

机器节点： 保留以前的方案

\#\# harproxy + keepalived 使用nginx负载均衡

10.126.88.11

10.126.88.12

\#\# harbor + kafka + opcenter 复用集群1

\#\# master 节点 ，集群名称：wcs_kubernetesv2_master

10.126.88.3 10.244.177.1 Apiserver,controller-manager,scheduler,etcd

10.126.88.5 10.244.179.1 Apiserver,controller-manager,scheduler,etcd

10.126.88.6 10.244.183.1 Apiserver,controller-manager,scheduler,etcd

\#\# 其他 node节点 （Kubelet,kube-proxy,docker）

10.126.82.211 10.244.147.1

10.126.90.75 10.244.149.1

10.126.82.228 10.244.161.1

10.126.90.82 10.244.167.1

10.126.124.36 10.244.173.1

10.126.90.70 10.244.175.1

10.126.88.10 10.244.181.1

**1.  搭建etcd**

**1.1.  安装etcd**

>   环境：centos7
>
>   可以使用 yum 安装 etcd: yum install -y etcd。
>
>   yum安装的etcd默认配置文件在/etc/etcd/etcd.conf，此配置需要修改，写成自己集群的配置,/usr/lib/systemd/system/etcd.service
>   修改etcd的启动参数配置，即调用上面的配置文件(注：/usr/lib/systemd/system/
>   这个目录存放systemctl启动的service，以下的service都会在此目录下)

使用cfssl生成etcd所需的密钥和证书

下载cfssl（该部分已失效，下载链接失效）

1. mkdir \~/bin

2. curl -s -L -o \~/bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64

3. curl -s -L -o \~/bin/cfssljson
   https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

4. chmod +x \~/bin/{cfssl,cfssljson}

5. export PATH=\$PATH:\~/bin

注：该部分的cfssl可从线上环境直接获取

**1.2  初始化证书颁发机构**

1. mkdir \~/cfssl

2. cd \~/cfssl

3. cfssl print-defaults config \> ca-config.json

4. cfssl print-defaults csr \> ca-csr.json


ca-config.json配置样例如下：
```
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "wcs": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
```

ca-csr.json 配置样例如下：
```
{
  "CN": "wcs",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "wcs",
      "OU": "cloudnative"
    }
  ]
}
```

注：3,4步可直接在线上的etcd机器上获取。


证书类介绍：

-   client certificate 用于通过服务器验证客户端。例如etcdctl，etcd
    proxy，fleetctl或docker客户端。

-   server certificate
    由服务器使用，并由客户端验证服务器身份。例如docker服务器或kube-apiserver。

-   peer certificate 由 etcd 集群成员使用，供它们彼此之间通信使用。

**1.3  配置证书**

（可直接在线上etcd机器上获取）

执行命令生成CA证书

cfssl gencert -initca ca-csr.json \| cfssljson -bare ca –

该步骤会生成ca-key.pem、ca.csr、ca.pem

将证书加入到本机的信用中心：

cat ca.pem \>\>/etc/pki/tls/certs/ca-bundle.crt

**1.4  配置etcd证书**

例：

```json
{
    "CN": "wcs",
    "hosts": [
        "127.0.0.1",
        "10.126.88.3",
        "10.126.88.5",
        "10.126.88.6",
        "etcd-node3",
        "etcd-node5",
        "etcd-node6",
        " 10.126.73.153"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "wcs",
            "OU": "cloudnative"
        }
    ]
}
```

将上述demo json保存为文件etcd-csr.json.


生成etcd证书：

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=wcs
etcd-csr.json \| cfssljson -bare etcd

注意：-profile选项指代etcd的集群名，根据实际情况设定。

**1.5  配置etcd使用证书启动**

将 CA证书 ca.pem 和 etcd 秘钥 etcd-key.pem ，以及 etcd 证书 etcd.pem
拷贝到各个节点的/etc/etcd/ssl 目录下，然后配置/etc/etcd/etcd.conf
配置文件，以10.126.88.3的配置为例，其他节点只需相应改变IP即可（注意下面配置是使用http和https两种客户端访问集群的方式，集群内部通信还是使用https的2380
端口 ，外部访问集群可以使用https的6544端口，也可以使用 4456 的 http 端口，
此处是为了兼容测试环境的客户后续可以升级只使用 https 访问方式）：

```
# [member]

ETCD_NAME=infra3

ETCD_DATA_DIR="/opt/wcs/etcd/etcd-node3"

ETCD_LISTEN_PEER_URLS="https://10.126.88.3:2380"

ETCD_LISTEN_CLIENT_URLS="https://0.0.0.0:6544,http://0.0.0.0:4456"

#[cluster]

ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.126.88.3:2380"

ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-v2"

ETCD_ADVERTISE_CLIENT_URLS=<https://10.126.88.3:6544,http://10.126.88.3:4456>
```

注：其他etcd节点使用该配置时需修改为相应的NAME和IP。

ps, etcd配置说明：
<pre>
name                          节点名称   
data-dir                      指定节点的数据存储目录   
listen-peer-urls              监听URL，用于与其他节点通讯   
listen-client-urls            对外提供服务的地址：比如 http://ip:2379,http://127.0.0.1:2379 ，客户端会连接到这里和 etcd 交互   
initial-advertise-peer-urls   该节点同伴监听地址，这个值会告诉集群中其他节点  
initial-cluster               集群中所有节点的信息，格式为 node1=http://ip1:2380,node2=http://ip2:2380,… 。注意：这里的 node1 是节点的 --name 指定的名字；后面的 ip1:2380 是 --initial-advertise-peer-urls 指定的值  
initial-cluster-state         新建集群的时候，这个值为 new ；假如已经存在的集群，这个值为 existing  
initial-cluster-token         创建集群的 token，这个值每个集群保持唯一。这样的话，如果你要重新创建集群，即使配置和之前一样，也会再次生成新的集群和节点 uuid；否则会导致多个集群之间的冲突，造成未知的错误  
advertise-client-urls         对外公告的该节点客户端监听地址，这个值会告诉集群中其他节点  
</pre>
**1.6  配置etcd.service**

```
[Unit]

Description=Etcd Server

After=network.target

After=network-online.target

Wants=network-online.target

Documentation=https://github.com/coreos

[Service]

Type=notify

WorkingDirectory=/opt/wcs/etcd/

EnvironmentFile=/etc/etcd/etcd.conf

ExecStart=/usr/bin/etcd \\

--name ${ETCD_NAME} \\

--cert-file=/etc/etcd/ssl/etcd.pem \\

--key-file=/etc/etcd/ssl/etcd-key.pem \\

--peer-cert-file=/etc/etcd/ssl/etcd.pem \\

--peer-key-file=/etc/etcd/ssl/etcd-key.pem \\

--trusted-ca-file=/etc/etcd/ssl/ca.pem \\

--peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \\

--initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \\

--listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \\

--listen-client-urls ${ETCD_LISTEN_CLIENT_URLS} \\

--advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \\

--initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \\

--initial-cluster
infra3=https://10.126.88.3:2380,infra5=https://10.126.88.5:2380,infra6=https://10.126.88.6:2380
\\

--initial-cluster-state new \\

--data-dir=\${ETCD_DATA_DIR}

Restart=on-failure

RestartSec=5

LimitNOFILE=65536

[Install]

WantedBy=multi-user.target
```


>   上面在启动参数中指定了etcd的工作目录和数据目录是/var/lib/etcd
>
>   \--cert-file和--key-file分别指定etcd的公钥证书和私钥
>
>   \--peer-cert-file和--peer-key-file分别指定了etcd的Peers通信的公钥证书和私钥。
>
>   \--trusted-ca-file指定了客户端的CA证书
>
>   \--peer-trusted-ca-file指定了Peers的CA证书
>
>   \--initial-cluster-state
>   new表示这是新初始化集群，--name指定的参数值必须在--initial-cluster中

**1.7  在另外两台节点修改相应的配置之后，启动etcd**

systemctl daemon-reload

systemctl enable etcd

systemctl start etcd

systemctl status etcd

注意：

\--initial-cluster
infra3=https://10.126.88.3:2380,infra5=https://10.126.88.5:2380,infra6=https://10.126.88.6:2380
\\

中配置的第一个节点，需要最后启动，否则会报错。

**1.8  检查集群是否健康，在任一节点运行**

**检测list是否正常：**

如果是http协议：

etcdctl member list

如果是https协议：

etcdctl --ca-file=/etc/etcd/ssl/ca.pem --cert-file=/etc/etcd/ssl/etcd.pem
--key-file=/etc/etcd/ssl/etcd-key.pem
--endpoints=https://10.126.88.3:6544,https://10.126.88.5:6544,https://10.126.88.6:6544
member list

如果使用了前端的负载均衡（例VIP为10.136.196.126）：

etcdctl --ca-file=/etc/etcd/ssl/ca.pem --cert-file=/etc/etcd/ssl/etcd.pem
--key-file=/etc/etcd/ssl/etcd-key.pem --endpoints=https://10.136.196.126:6544
cluster-health

其中member list和cluster-health可以互换

在82和84机器上搭建相同的ETCD服务，启动。

可以看到集群各个节点的状态，到此带有证书验证的etcd集群搭建完成。

**检测数据是否正常：**

在集群内任意节点上：

curl <http://10.126.88.3:4456/v2/keys/hxx>

curl <http://10.126.88.5:4456/v2/keys/hxx>

curl <http://10.126.88.6:4456/v2/keys/hxx>

插入数据：

curl http:// 10.126.88.3:4456/v2/keys/hxx -XPUT -d value="hxxworld"

curl http:// 10.126.88.5:4456/v2/keys/hxx -XPUT -d value="hxxworld"

curl http:// 10.126.88.6:4456/v2/keys/hxx -XPUT -d value="hxxworld"

**1.9  Q&A：**

>1.  **etcd[10365]: request sent was ignored**
>
>   处理方法：清空etcd目录（\${ WorkingDirectory
>   }）下的文件，关闭etcd服务，再重新加载，重启。
>2.  **x509: certificate signed by unknown authority**
>
>   处理方法：这个错误是由于自建的ca证书不被本机信任导致的，只要把ca证书导入到本地信任中心即可。具体只要在每个主机主机上运行一下：
>
>   cat ca证书文件 \>\> /etc/pki/tls/certs/ca-bundle.crt 即可。
>
>   **3．问题：member 7bda1219b4979150 has already been bootstrapped**
>
>    remove the old member via dynamic configuration API
>
>    add the new member via dynamic configuration API
>
>    remove all residuals datas : \`rm -rf /var/lib/etcd2/\*\`
>
>    start etcd with initial-cluster=existing
>
>   <https://github.com/coreos/etcd/issues/2780>
>
>   重启集群内其他节点。
>
>   **4.执行etcdctl --debug member list报错：**
>
>   Error: client: etcd cluster is unavailable or misconfigured; error \#0: dial tcp
>   127.0.0.1:2379: getsockopt: connection refused
>
>   ; error \#1: dial tcp 127.0.0.1:4001: getsockopt: connection refused
>
>   该问题由于测试端口是默认的2379、4001，但etcd配置的端口是4456，手动加参数etcdctl
>   --debug --endpoints http://127.0.0.1:4456 member list
>
>   **5，Job kubelet.service/start failed with result 'dependency'**
>
>   Journalctl -ex 查看日志，关注.service中的配置是否正确。
>
>   **6. systemd启动etcd服务的时候出现错误：Failed at step CHDIR spawning
>   /usr/bin/etcd: No such file or directory**
>
>   解决办法：etcd.service服务配置文件中设置的工作目录WorkingDirectory=/var/lib/etcd/必须存在，否则会报以上错误
>
>   **7，could not get cluster response from https://192.168.183.169:2380**
>
>   解决方法：查看集群内机器的证书文件是否一致。
>
>   **8，request sent was ignored (cluster ID mismatch**
>
>   解决方法：
>
>   处理方法：清空etcd目录（\${ WorkingDirectory
>   }）下的文件，关闭etcd服务，再重新加载，重启。
>
>   **9,{"code":5100,"message":"Invalid policy: no key usage available"}
>
>  NoKeyUsages indicates that the profile does not permit any
>	// key usages for the certificate.
>
>   解决方法。

**2.  搭建HAproxy+keepalived**

这个模块相当于架构图中的Load
Balancer模块，haproxy+keepalived。构建两台主备关系的负载均衡集群，haproxy负责流量转发，keepalived提供对外提供服务的VIP，并且监控HAproxy的状态，当主机发生故障时，VIP会漂移到备机上继续提供服务，当主机恢复之后再漂移回来，形成高可用的服务。

**2.1  搭建 HAproxy（两台机器配置相同）**

**2.1.1  下载**

可以去http://www.haproxy.org/ 下载安装包

**2.1.2  安装**

```bash
# tar zxvf haproxy-1.7.2.tar.gz
# cd haproxy-1.7.2
# make PREFIX=/root/haproxy TARGET=linux2628
# make install PREFIX=/root/haproxy]
```

**2.1.3  配置**

\# cd haproxy

\# mkdir conf

\# touch haproxy.cfg

编辑haproxy.cfg，内容如下：

global \#全局属性

daemon \#以daemon方式在后台运行

maxconn 4096 \#最大同时连接

pidfile /root/haproxy/conf/haproxy.pid \#指定保存HAProxy进程号的文件

log 127.0.0.1 local0 notice

log 127.0.0.1 local1 warning

defaults \#默认参数

mode tcp \#ssl 使用tcp

timeout connect 5000ms \#连接server端超时5s

timeout client 50000ms \#客户端响应超时50s

timeout server 50000ms \#server端响应超时50s

stats uri /stats \#统计页面url

stats refresh 30s \#统计页面自动刷新时间

log global

listen k8s-http-in

bind \*:3218 \#监听80端口

mode tcp

balance roundrobin \#使用RR负载均衡算法

server k8s-http-server1 10.126.88.3:3218 \#k8s-apiserver http访问端口

server k8s-http-server2 10.126.88.5:3218

server k8s-http-server3 10.126.88.6:3218

listen k8s-https-in

bind \*:8123 \#监听443端口，使用SSL穿透，haproxy只做tcp流量转发，https在后端验证

mode tcp

balance roundrobin \#使用RR负载均衡算法

server k8s-https-server1 10.126.88.3:8123 \#k8s-apiserver https访问端口

server k8s-https-server2 10.126.88.5:8123

server k8s-https-server3 10.126.88.6:8123

listen etcd-http-in

bind \*:4456 \#监听80端口

mode tcp

balance roundrobin \#使用RR负载均衡算法

server etcd-http-server1 10.126.88.3:4456 \#etcd http访问端口

server etcd-http-server2 10.126.88.5:4456

server etcd-http-server3 10.126.88.6:4456

listen etcd-https-in

bind \*:6544 \#监听80端口

mode tcp

balance roundrobin \#使用RR负载均衡算法

server etcd-https-server1 10.126.88.3:6544 \#etcd https访问端口

server etcd-https-server2 10.126.88.5:6544

server etcd-https-server3 10.126.88.6:6544

**2.1.4  将 HAproxy注册成系统服务**

在 /etc/init.d 目录下添加 HAProxy 服务的启停脚本：

\# vi /etc/init.d/haproxy

脚本内容如下：

\#!/bin/sh

set -e

PATH=/sbin:/bin:/usr/sbin:/usr/bin:/home/ha/haproxy/sbin

PROGDIR=/root/haproxy

PROGNAME=haproxy

DAEMON=\$PROGDIR/sbin/\$PROGNAME

CONFIG=\$PROGDIR/conf/\$PROGNAME.cfg

PIDFILE=\$PROGDIR/conf/\$PROGNAME.pid

DESC="HAProxy daemon"

SCRIPTNAME=/etc/init.d/\$PROGNAME

\# Gracefully exit if the package has been removed.

test -x \$DAEMON \|\| exit 0

start()

{

echo -e "Starting \$DESC: \$PROGNAME\\n"

\$DAEMON -f \$CONFIG

echo "."

}

stop()

{

echo -e "Stopping \$DESC: \$PROGNAME\\n"

haproxy_pid="\$(cat \$PIDFILE)"

kill \$haproxy_pid

echo "."

}

restart()

{

echo -e "Restarting \$DESC: \$PROGNAME\\n"

\$DAEMON -f \$CONFIG -p \$PIDFILE -sf \$(cat \$PIDFILE)

echo "."

}

case "\$1" in

start)

start

;;

stop)

stop

;;

restart)

restart

;;

\*)

echo "Usage: \$SCRIPTNAME {start\|stop\|restart}" \>&2

exit 1

;;

esac

exit 0

**2.1.5 将启动脚本添加可执行权限**

\# chmod +x haproxy

**2.1.6 配置日志**

HAProxy 不会直接输出文件日志，需要借助 Linux 的 rsyslog 来让 HAProxy 输出日志。
之前配置的haproxy.cfg中：

global

...

log 127.0.0.1 local0 info

log 127.0.0.1 local1 warning

...

defaults

...

log global

...

意思是将info级（及以上）的日志推送到rsyslog的local0 接口，将 warn
级（及以上）的日志推送到 rsyslog 的 local1 接口，并且所有 frontend 都默认使用
global 中的日志配置。

注意：info 级的日志会打印 HAProxy
处理的每一条请求，会占用很大的磁盘空间，在生产环境中，建议将日志级别调整为
notice。

**2.1.7 为 rsyslog 添加 haproxy日志的配置**

**\# vi /etc/rsyslog.d/haproxy.conf**

配置文件内容如下：

\$ModLoad imudp

\$UDPServerRun 514

\$FileCreateMode 0644 \#日志文件的权限

\$FileOwner root \#日志文件的owner

local0.\* /var/log/haproxy.log \#local0接口对应的日志输出文件

local1.\* /var/log/haproxy_warn.log \#local1接口对应的日志输出文件

**之后修改 rsyslog 的启动参数**

\# vi /etc/sysconfig/rsyslog

配置文件内容如下：

\# Options for rsyslogd

\# Syslogd options are deprecated since rsyslog v3.

\# If you want to use them, switch to compatibility mode 2 by "-c 2"

\# See rsyslogd(8) for more details

SYSLOGD_OPTIONS="-c 2 -r -m 0"

之后重启 rsyslog

\# service rsyslog restart

**2.1.8 用 logrotate进行日志切分**

通过 rsyslog 输出的日志是不会切分的，所以需要通过Linux提供的 logrotate
来对日志文件进行切分。

使用 root 用户，创建 haproxy 日志切分配置文件：

\# mkdir /root/logrotate

\# vi /root/logrotate/haproxy

配置文件内容如下：

/var/log/haproxy.log /var/log/haproxy_warn.log { \#切分的两个文件名

daily \#按天切分

rotate 7 \#保留7份

create 0644 root root \#创建新文件的权限、用户、用户组

compress \#压缩旧日志

delaycompress \#延迟一天压缩

missingok \#忽略文件不存在的错误

dateext \#旧日志加上日志后缀

sharedscripts \#切分后的重启脚本只运行一次

postrotate \#切分后运行脚本重载rsyslog，让rsyslog向新的日志文件中输出日志

/bin/kill -HUP \$(/bin/cat /var/run/syslogd.pid 2\>/dev/null) &\>/dev/null

endscript

}

**然后将 logrotate 配置在 crontab 中：**

0 0 \* \* \* /usr/sbin/logrotate /root/logrotate/haproxy

启动haproxy

启动命令：

启动： \# service haproxy start

重启： \# service haproxy restart

停止： \# service haproxy stop

**2.1.9 测试**

可以先查看进程是否启动，端口是否占用，和相应的日志（/var/log/haproxy.log）内容。转发测试可以搭建TOMCAT服务或者NGINX服务进行流量的转发。

**2.2  搭建keepalive**

利用 Keepalived 实现 HAProxy 的热备方案，即两台主机上的 HAProxy
实例同时运行，其中全总较高的实例为
MASTER，MASTER出现异常时，另一台主机上的实例自动接管所有的流量。

在运行着 HAProxy 实例的两台主机上分别运行着 Keepalived 实例，这两个 Keepalived
争抢同一个虚IP地址，两个HAProxy也尝试着去绑定同一个虚 IP
地址上的端口。只能有一个 Keepalived 能抢到这个虚 IP，抢到这个虚 IP
Keepalived的主机上的 HAProxy 即为当前的 MASTER。

Keepalived 内部维护一个权重值，权重值最高的 Keepalived 实例能够抢到虚
IP。Keepalived 会定期检查所在主机上的 HAProxy 的状态，状态健康时，则权重值增加。

**2.2.1 下载**

官网下载安装包http://www.keepalived.org/keepalived-1.3.4.tar.gz

**2.2.2 安装插件**

**2.2.2.1 安装psmisc**

使用 killall -0 检查 HAProxy 服务是否存在。如果没有
killall 命令，则 需要安装 psmisc 包。

\# yum install psmisc

**2.2.2.2 安装openssl**

\# yum install openssl-devel

**2.2.2.3 安装libnl/libnl-3库**

\# yum install libnl\*

**2.2.2.4 安装libnfnetlink-devel**

\# yum install libnfnetlink-devel

**2.2.3 安装与配置** keepalived

**2.2.3.1 安装**

\# tar zxvf keepalived-1.3.4.tar.gz

\# cd keepalived-1.3.4

\# ./configure --prefix=/usr/local/keepalived

\# make

\# make install

**2.2.3.2 配置**

将keepalived注册为系统服务：

\# cp /usr/local/keepalived/sbin/keepalived /usr/sbin

\# cp /usr/local/keepalived/etc/sysconfig/keepalived /etc/sysconfig/

\# touch /etc/init.d/keepalived

\# chmod +x /etc/init.d/keepalived

\# vi /etc/init.d/keepalived

/etc/init.d/keepalived 的文件内容：

\#!/bin/sh

\#

\# Startup script for the Keepalived daemon

\#

\# processname: keepalived

\# pidfile: /var/run/keepalived.pid

\# config: /etc/keepalived/keepalived.conf

\# chkconfig: - 21 79

\# description: Start and stop Keepalived

\# Source function library

. /etc/rc.d/init.d/functions

\# Source configuration file (we set KEEPALIVED_OPTIONS there)

. /etc/sysconfig/keepalived

RETVAL=0

prog="keepalived"

start() {

echo -n \$"Starting \$prog: "

daemon keepalived \${KEEPALIVED_OPTIONS}

RETVAL=\$?

echo

[ \$RETVAL -eq 0 ] && touch /var/lock/subsys/\$prog

}

stop() {

echo -n \$"Stopping \$prog: "

killproc keepalived

RETVAL=\$?

echo

[ \$RETVAL -eq 0 ] && rm -f /var/lock/subsys/\$prog

}

reload() {

echo -n \$"Reloading \$prog: "

killproc keepalived -1

RETVAL=\$?

echo

}

\# See how we were called.

case "\$1" in

start)

start

;;

stop)

stop

;;

reload)

reload

;;

restart)

stop

start

;;

condrestart)

if [ -f /var/lock/subsys/\$prog ]; then

stop

start

fi

;;

status)

status keepalived

RETVAL=\$?

;;

\*)

echo "Usage: \$0 {start\|stop\|reload\|restart\|condrestart\|status}"

RETVAL=1

esac

exit \$RETVAL

编辑配置文件

\# mkdir /etc/keepalived/

\# cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/

\# vi /etc/keepalived/keepalived.conf

配置文件内容：

global_defs {

router_id LVS_DEVEL \#虚拟路由名称

}

\#HAProxy健康检查配置

vrrp_script chk_haproxy {

script "/usr/bin/killall -0 haproxy" \#使用killall
-0检查haproxy实例是否存在，性能高于ps命令,注意此处需要使用绝对路径，否则可能会无法执行killall命令

interval 2 \#脚本运行周期

weight 2 \#每次检查的加权权重值

}

\#虚拟路由配置

vrrp_instance VI_1 {

state BACKUP \#本机实例状态，MASTER/BACKUP，备机配置文件中请写BACKUP

interface eth1 \#本机网卡名称，使用ifconfig命令查看

virtual_router_id 51 \#虚拟路由编号，主备机保持一致

priority 100 \#本机初始权重，备机请填写小于主机的值（例如100）

advert_int 1 \#争抢虚地址的周期，秒

virtual_ipaddress {

10.136.196.126 dev eth1

}

track_script {

chk_haproxy \#对应的健康检查配置

}

}

这里的注意事项：

 interface eth1 网卡名称必须正确，确定是提供服务的网卡


对外提供服务的虚拟IP需要向运维申请，添加相应的路由才可以被同网段的其他节点访问到

2.2.3.3 启动

启动：\# service keepalived start

重启：\# service keepalived restart

停止：\# service keepalived stop

2.2.3.4 测试

首先查看进程是否正常启动，之后查看虚拟IP是否绑定到了物理网卡上（ip addr
show）主备两台机器正常情况下是主机绑定了VIP，日志（/var/log/messages），还可以测试服务是否高可用。

**3.  Kubernetes集群配置**

**3.1  Kubernetes各组件TLS证书和密钥**

我们目前同时开发kube-apiserver的https端口和http端口，集群内部访问
apiserver使用https端口，集群外部访问可以选择。启用Kubernetes
集群相关组件的TLS通信和双向认证，下面将使用工具生成各组件TL 的证书和私钥。

**3.1.1  CA证书和私钥**

我们88.3机器上/root/cfssl-online/kubernetes目录下创建 k8s
集群需要的证书和私钥。复用etcd的ca-config.json，内容如下：

>   {

>   "signing": {

>   "default": {

>   "expiry": "87600h"

>   },

>   "profiles": {

>   "wcs": {

>   "usages": ["signing", "key encipherment", "server auth", "client auth"],

>   "expiry": "87600h"

>   }

>   }

>   }

>   }

>   创建 CA 证书签名请求配置 ca-csr.json ：

>   {

>   "CN": "Kubernetes",

>   "key": {

>   "algo": "rsa",

>   "size": 2048

>   },

>   "names": [

>   {

>   "C": "CN",

>   "ST": "BeiJing",

>   "L": "BeiJing",

>   "O": "k8s",

>   "OU": "cloudnative"

>   }

>   ]

>   }

-   CN即Common Name，kube-apiserver从证书中提取该字段作为请求的用户名

-   O即Organization，kube-apiserver 从证书中提取该字段作为请求用户所属的组

>   下面使用 cfss 生成 CA 证书和私钥：

>   cfssl gencert -initca ca-csr.json \| cfssljson -bare ca

>   ca-key.pem 和 ca.pem后期会用到

**3.1.2  创建kube-apiserver的证书和秘钥**

创建kube-apiserver证书签名请求配置apiserver-csr.json

{

"CN": "kubernetes",

"hosts": ["127.0.0.1",

"10.126.88.3",

"10.126.88.5",

"10.126.88.6",

"10.96.0.1",

"kubernetes",

"kubernetes.default",

"kubernetes.default.svc",

"kubernetes.default.svc.cluster",

"kubernetes.default.svc.cluster.local",

"10.136.196.126",

"10.126.73.153",

"wcs.k8sv2-etcd.58dns.org",

"wcs.k8sv2-k8sapiserver.58dns.org"

],

"key": {

"algo": "rsa",

"size": 2048

},

"names": [{

"C": "CN",

"ST": "BeiJing",

"L": "BeiJing",

"O": "k8s",

"OU": "cloudnative"

}]

}

注意上面配置 hosts 字段中制定授权使用该证书的IP
和域名列表，因为现在要生成的证书需要 Kubernetes
Master集群各个节点使用，所以这里指定了集群各个节的 IP 和
hostname。同时还要指定集群内部 kube-apiserver的多个域名和 IP 地址 10.96.0.1
(后边
kube-apiserver-service-cluster-ip-range=10.96.0.0/12参数的指定网段第一个IP) 。

注意: 在以前的文档中这个配置叫
kubernetes-csr.json，为了明确划分职责，这个证书目前被重命名以表示其专属于
apiserver 使用；加了一个 \*.kubernetes.master 域名以便内部私有 DNS
解析使用(可删除)；至于很多人问过 kubernetes
这几个能不能删掉，答案是不可以的；因为当集群创建好后，default namespace
下会创建一个叫 kubenretes 的 svc，有一些组件会直接连接这个 svc 来跟 api
通讯的，证书如果不包含可能会出现无法连接的情况；其他几个 kubernetes
开头的域名作用相同

下面生成kube-apiserver的证书和私钥：

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=wcs
apiserver-csr.json \| cfssljson -bare apiserver

**3.1.3  创建kube-controller-manager客户端证书和私钥**

下面创建kube-controller-manager 客户端访问 ApiServer
所需的证书和私钥。controller-manager证书的签名请求配置controller-manager-csr.json：

>   {

>   "CN": "system:kube-controller-manager",

>   "hosts": [

>   "10.126.88.3",

>   "10.126.88.5",

>   "10.126.88.6"

>   ],

>   "key": {

>   "algo": "rsa",

>   "size": 2048

>   },

>   "names": [{

>   "C": "CN",

>   "ST": "BeiJing",

>   "L": "BeiJing",

>   "O": "system:kube-controller-manager",

>   "OU": "cloudnative"

>   }]

>   }

kube-apiserver 将提取 CN
作为客户端的用名，这里是system:kube-controller-manager。kube-apiserver预定义RBAC使用的
ClusterRoleBindings system:kube-controller-manager将用户
system:kube-controller-manager 与 ClusterRole system:kube-controller-manager
绑定。

下面生成证书和私钥：

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=wcs
controller-manager-csr.json \| cfssljson -bare controller-manager

**3.1.4  创建kube-schedule客户端证书和私钥**

下面创建 kube-scheduler 客户端访问 ApiServer 所需的证书和私钥。
scheduler证书的签名请求配置 scheduler-csr.json ：

>   {

>   "CN": "system:kube-scheduler",

>   "hosts": [

>   "10.126.88.3",

>   "10.126.88.5",

>   "10.126.88.6"

>   ],

>   "key": {

>   "algo": "rsa",

>   "size": 2048

>   },

>   "names": [{

>   "C": "CN",

>   "ST": "BeiJing",

>   "L": "BeiJing",

>   "O": "system:kube-scheduler",

>   "OU": "cloudnative"

>   }]

>   }

kube-scheduler将提取CN作为客户端的用户名，这里是system:kube-scheduler。kube-apiserver预定义的RBAC使用的ClusterRoleBindings
system:kube-scheduler将用户system:kube-scheduler与ClusterRole
system:kube-scheduler绑定。

下面生成证书和私钥：

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=wcs
scheduler-csr.json \| cfssljson -bare scheduler

**3.2  kubernetes master集群部署**

部署的Master节点集群由88.3，88.5,88.6
三个节点组成，每上部署kube-apiserver,kube-controller-manager,kube-scheduler三个核心组件。
kube-apiserver 的 3个实例同时提供服务，在其前端部署一高可用的负载均衡器作为
kube-apiserver的地址。kube-controller-manager 和 kube-scheduler
也是各自3个实例，在同一时刻只能有1个,这个实例通过选举产生。

**3.2.1  安装kubernetes-master.x86_64**

我们可以使用 yum 来安装kubernetes 的相关组件，但是 yum 安装的版本 较低，不是我们想要使用1.12.4版本 ，因此我们可以利用yum 来生成 service 的系统配置文件，先下载kubernetes1.12.4的压缩包，解压之后，用 /opt/kubernetes-v1.12.4/server/kubernetes/server/bin目录下的相关模块替换/usr/bin/目录下的kube-apiserver、kube-controller-manager、kube-scheduler 和 kubectl ，这样 ，这样就可以使用1.12.4 的 k8s 相关模块了。

Yum 命令：

yum install -y kubernete-master.x86_64

安装包下载 ：

<https://github.com/kubernetes/kubernetes/releases/tag/v1.12.4>
10.136.196.207上，已经存在了该版本的安装包。
找到相应版本下载压缩包即可

将前面生成的ca.pem,apiserver-key.pem,apiserver.pem,controller-manager.pem,controller-manager-key.pem,scheduler-key.pem,scheduler.pem
拷贝到各个节点的/etc/kubernetes/pki目录下：

mkdir -p /etc/kubernetes/pki

cp
{ca.pem,apiserver-key.pem,apiserver.pem,admin.pem,admin-key.pem,controller-manager.pem,controller-manager-key.pem,scheduler-key.pem,
scheduler.pem} /etc/kubernetes/pki

**3.2.2  kube-apiserver部署**

修改 /usr/lib/systemd/system/kube-apiserver.service:

>   [Unit]

>   Description=Kubernetes API Server

>   Documentation=https://github.com/GoogleCloudPlatform/kubernetes

>   After=network.target

>   After=etcd.service

>   [Service]

>   EnvironmentFile=-/etc/kubernetes/config

>   EnvironmentFile=-/etc/kubernetes/apiserver

>   ExecStart=/usr/bin/kube-apiserver \\

>   \--logtostderr=false \\

>   \--v=2 \\

>   \--log-dir=/opt/wcs/kubernetes/log/apiserver \\

>   \--advertise-address=10.126.88.3 \\

>   \--bind-address=0.0.0.0 \\

>   \--secure-port=8123 \\

>   \--insecure-bind-address=0.0.0.0 \\

>   \--insecure-port=3218 \\

>   \--allow-privileged=true \\

>   \--etcd-servers=https://10.126.88.3:6544,https://10.126.88.5:6544,https://10.126.88.6:6544
>   \\

>   \--etcd-cafile=/etc/etcd/ssl/ca.pem \\

>   \--etcd-certfile=/etc/etcd/ssl/etcd.pem \\

>   \--etcd-keyfile=/etc/etcd/ssl/etcd-key.pem \\

>   \--storage-backend=etcd3 \\

>   \--service-cluster-ip-range=10.96.0.0/12 \\

>   \--tls-cert-file=/etc/kubernetes/pki/apiserver.pem \\

>   \--tls-private-key-file=/etc/kubernetes/pki/apiserver-key.pem \\

>   \--client-ca-file=/etc/kubernetes/pki/ca.pem \\

>   \--service-account-key-file=/etc/kubernetes/pki/ca-key.pem \\

>   \--kubelet-client-certificate=/etc/kubernetes/pki/admin.pem \\

>   \--kubelet-client-key=/etc/kubernetes/pki/admin-key.pem \\

>   \--experimental-bootstrap-token-auth=true \\

>   \--apiserver-count=3 \\

>   \--enable-swagger-ui=true \\

>   \--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds
>   \\

>   \--authorization-mode=RBAC \\

>   \--audit-log-maxage=30 \\

>   \--audit-log-maxbackup=3 \\

>   \--audit-log-maxsize=100 \\

>   \--audit-log-path=/opt/wcs/kubernetes/log/apiserver/apiserver.log

>   Restart=on-failure

>   Type=notify

>   LimitNOFILE=65536

>   [Install]

>   WantedBy=multi-user.target

\--insecure-port 指定http协议不安全端口

\--secure-port指定https安全端口，kube-scheduler、kube-controller-manager、kubelet、kube-proxy、kubectl等组件都将使用安全端口与ApiServer通信(实际上会由我们在前端部署的负载均衡器代理)。

\--authorization-mode=RBAC表示在安全端口启用RBAC授权模式，在授权过程会拒绝会授权的请求。kube-scheduler、kube-controller-manager、kubelet、kube-proxy、kubectl等组件都使用各自证书或kubeconfig指定相关的User、Group来通过RBAC授权。

\--admission-control为准入机制，这里配置了NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds

\--service-cluster-ip-range指定Service Cluster
IP的地址段，注意该地址段是Kubernetes
Service使用的，是虚拟IP，从外部不能路由可达。

>   启动 kube-apiserver:

>    systemctl daemon-reload

>    systemctl enable kube-apiserver

>    systemctl start kube-apiserver

>    systemctl status kube-apiserver

**3.2.3  创建kubernetes-admin客户端证书和私钥**

创建admin证书的签名请求配置admin-csr.json：

{

"CN": "kubernetes-admin",

"hosts": [

"127.0.0.1",

"10.126.88.3",

"10.126.88.5",

"10.126.88.6",

"etcd-node3",

"etcd-node5",

"etcd-node6",

" 10.126.73.153"

],

"key": {

"algo": "rsa",

"size": 2048

},

"names": [

{

"C": "CN",

"ST": "BeiJing",

"L": "BeiJing",

"O": "system:master",

"OU": "cloudnative"

}

]

}

kube-apiserver将提取CN作为客户端的用户名，这里是kubernetes-admin，将提取O作为用户所属的组，这里是system:master。kube-apiserver预定义了一些RBAC使用的ClusterRoleBindings，例如cluster-admin将组system:masters与ClusterRole
cluster-admin绑定，而cluster-admin拥有访问kube-apiserver的所有权限，因此kubernetes-admin这个用户将作为集群的超级管理员。

下面生成kubernetes-admin的证书和私钥：

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=wcs
admin-csr.json \| cfssljson -bare admin

**3.2.4  配置 kubectl访问apiserver**

我们已经部署了kube-apiserver，并且前面已经有kubernetes-admin的证书和私钥，我们将使用这个户访问ApiServer。

kubectl --server=https://10.126.73.153:8123
--certificate-authority=/etc/kubernetes/pki/ca.pem
--client-certificate=/etc/kubernetes/pki/admin.pem
--client-key=/etc/kubernetes/pki/admin-key.pem get componentstatuses

根据返回结果，我们会发现 kube-scheduler 和 controller-manager
是不健康的，因为还没有部署，etcd 集群处于非健康状态。但是使用kubectl
时需要指定ApiServer的地址以及客户端证书，用起来比较繁琐。接下来我们创建kubernetes-admin的kubeconfig文件admin.conf。

我们可以编写admin.sh 脚本，来生成配置文件 ,脚本内容如下：

>   cd /etc/kubernetes

>   export KUBE_APISERVER="https://10.126.73.153:8123"

>   \# set-cluster

>   kubectl config set-cluster kubernetes \\

>   \--certificate-authority=/etc/kubernetes/pki/ca.pem \\

>   \--embed-certs=true \\

>   \--server=\${KUBE_APISERVER} \\

>   \--kubeconfig=admin.conf

>   \# set-credentials

>   kubectl config set-credentials kubernetes-admin \\

>   \--client-certificate=/etc/kubernetes/pki/admin.pem \\

>   \--embed-certs=true \\

>   \--client-key=/etc/kubernetes/pki/admin-key.pem \\

>   \--kubeconfig=admin.conf

>   \# set-context

>   kubectl config set-context kubernetes-admin\@kubernetes \\

>   \--cluster=kubernetes \\

>   \--user=kubernetes-admin \\

>   \--kubeconfig=admin.conf

>   \# set default context

>   kubectl config use-context kubernetes-admin\@kubernetes \\

>   \--kubeconfig=admin.conf

执行脚本 之后，在 /etc/kubernetes 目录 下会生成相应的配置文件

为了使用kubectl访问apiserver使用admin.conf，将其拷贝到/root/.kube中并重命名成config：

cp /etc/kubernetes/admin.conf \~/.kube/config

之后就可以直接使用kubectl来访问apiserver了（其他master节点和node节点的config都可以使用这份配置，只需要把config拷贝到各个节点的/root/.kube/目录下即可。如果没有/root/.kube/目录，可以先使用带证书的kubectl查询一遍）

**3.2.5  kube-apiserver高可用**

经过前面的步骤，我们已在10.126.88.3/5/6上部署了Kubernetes Master 节点的
kube-apiserver，地址如下：

10.126.88.3:8123

10.126.88.5:8123

10.126.88.6:8123

为了实现高可用，以在前面放一个负载均衡器来代理访问kube-apiserver的请求，再对负载均衡器实现高可用。参考我们上面搭建的HAproxy+keepalived，VIP
为10.126.73.153，端口8123，因此我们之前的/root/.kube/config使用的是这个VIP:port

**3.2.6  kube-controller-manager部署**

前面已经创建了controller-manager.pem和controller-manage-key.pem，下面生成，下面生成controller-manager的kubeconfig文件controller-manager.conf，同样创建脚本controller-manager.sh生成配置:

>   cd /etc/kubernetes

>   export KUBE_APISERVER="https://10.126.73.153:8123"

>   \# set-cluster

>   kubectl config set-cluster kubernetes \\

>   \--certificate-authority=/etc/kubernetes/pki/ca.pem \\

>   \--embed-certs=true \\

>   \--server=\${KUBE_APISERVER} \\

>   \--kubeconfig=controller-manager.conf

>   \# set-credentials

>   kubectl config set-credentials system:kube-controller-manager \\

>   \--client-certificate=/etc/kubernetes/pki/controller-manager.pem \\

>   \--embed-certs=true \\

>   \--client-key=/etc/kubernetes/pki/controller-manager-key.pem \\

>   \--kubeconfig=controller-manager.conf

>   \# set-context

>   kubectl config set-context system:kube-controller-manager\@kubernetes \\

>   \--cluster=kubernetes \\

>   \--user=system:kube-controller-manager \\

>   \--kubeconfig=controller-manager.conf

>   \# set default context

>   kubectl config use-context system:kube-controller-manager\@kubernetes
>   --kubeconfig=controller-manager.conf

之后修改/usr/lib/systemd/system/kube-controller-manager.service文件 ：

>   [Unit]

>   Description=Kubernetes Controller Manager

>   Documentation=https://github.com/GoogleCloudPlatform/kubernetes

>   [Service]

>   EnvironmentFile=-/etc/kubernetes/config

>   EnvironmentFile=-/etc/kubernetes/controller-manager

>   ExecStart=/usr/bin/kube-controller-manager \\

>   \--logtostderr=false \\

>   \--v=2 \\

>   \--master=https://10.126.73.153:8123 \\

>   \--kubeconfig=/etc/kubernetes/controller-manager.conf \\

>   \--cluster-name=kubernetes \\

>   \--cluster-signing-cert-file=/etc/kubernetes/pki/ca.pem \\

>   \--cluster-signing-key-file=/etc/kubernetes/pki/ca-key.pem \\

>   \--service-account-private-key-file=/etc/kubernetes/pki/ca-key.pem \\

>   \--root-ca-file=/etc/kubernetes/pki/ca.pem \\

>   \--insecure-experimental-approve-all-kubelet-csrs-for-group=system:bootstrappers
>   \\

>   \--use-service-account-credentials=true \\

>   \--service-cluster-ip-range=10.96.0.0/12 \\

>   \--cluster-cidr=10.244.0.0/16 \\

>   \--allocate-node-cidrs=true \\

>   \--leader-elect=true \\

>   \--log-dir=/opt/wcs/kubernetes/log/controller-manager \\

>   \--controllers=\*,bootstrapsigner,tokencleaner

>   Restart=on-failure

>   LimitNOFILE=65536

>   [Install]

>   WantedBy=multi-user.target

>   其中

>   \--service-cluster-ip-range 和 kube-api-server 中指定的参数值一致

>   \--logtostderr=false 设置日志不输出到标准错误中

>   在各节点上启动:

>   systemctl daemon-reload

>   systemctl enable kube-controller-manager

>   systemctl start kube-controller-manager

>   systemctl status kube-controller-manager

到这一步三个Master节点上的kube-controller-manager部署完成，通过选举出一个leader工作。
这里我们重新指定了k8s各个模块的日志目录，并且不输出到标准错误中取，因此我们可以查看/opt/wcs/kubernetes/log/下各个模块的日志，查看三个节点的controller-manager的INFO日志，其中只有一个节点在工作，其他两个节点在等待选举leader，表示controller-manager部署成功。

**3.2.7  kube-scheduler部署**

前面已经创建了scheduler.pem
和scheduler-key，下面生成kube-scheduler的kubeconfig文件scheduler.conf，编写脚本scheduler.sh并运行生成配置：

>   cd /etc/kubernetes

>   export KUBE_APISERVER="https://10.126.73.153:8123"

>   \# set-cluster

>   kubectl config set-cluster kubernetes \\

>   \--certificate-authority=/etc/kubernetes/pki/ca.pem \\

>   \--embed-certs=true \\

>   \--server=\${KUBE_APISERVER} \\

>   \--kubeconfig=scheduler.conf

>   \# set-credentials

>   kubectl config set-credentials system:kube-scheduler \\

>   \--client-certificate=/etc/kubernetes/pki/scheduler.pem \\

>   \--embed-certs=true \\

>   \--client-key=/etc/kubernetes/pki/scheduler-key.pem \\

>   \--kubeconfig=scheduler.conf

>   \# set-context

>   kubectl config set-context system:kube-scheduler\@kubernetes \\

>   \--cluster=kubernetes \\

>   \--user=system:kube-scheduler \\

>   \--kubeconfig=scheduler.conf

>   \# set default context

>   kubectl config use-context system:kube-scheduler\@kubernetes
>   --kubeconfig=scheduler.conf

>   之后修改/usr/lib/systemd/system/kube-scheduler.service

>   在各节点上启动：

>   systemctl daemon-reload

>   systemctl enable kube-scheduler

>   systemctl start kube-scheduler

>   systemctl status kube-scheduler

同样观察相应日志，三个节点只有一个leader在执行工作。

至此Master节点模块全部署完成，使用kubectl get cs查看各个核心组件的状态。

**4.  搭建node集群**

Kubernetes的一个Node节点上需要运行如下组件：

-   Docker

-   kubelet

-   kube-proxy

Node节点可以使用yum进行安装，yum install -y
kubernetes-node-\*，安装之后会自动生成相应的service文件，但是使用yum安装的模块版本过低，所以需要将之前下载的kubernetes压缩包解压，然后使用其中/kubernetes/server/bin目录下的kubelet、kube-proxy、kubectl将/usr/bin/目录下的对应文件进行替换，使用1.8.13的版本。

Kubernetesv.1.8.13在10.136.196.207环境上已下载1.8.13版本，下载链接如下：https://github.com/kubernetes/kubernetes/releases/tag/v1.8.13

**4.1  Doker安装配置**

Docker可以使用yum进行安装，yum –y install docker
,目前yum安装的docker版本是稳定的1.12.6,如果想升级版本，可以下载相应的压缩包然后替换/usr/bin/目录下的docker二进制运行文件。安装完成之后，修改/usr/lib/systemd/system/docker.service文件，增加一个国内的加速器--registry
-mirror=https://jxus37ad.mirror.aliyuncs.com，然后指定一个--
bip=10.244.0.1/24，注意，这个bip的网段要和之后安装 flannel
网段一致，先手动指定一个，然后根据 flannel 分配的网段进行修改，另外还要指定
docker私服的配置，具体方式可以参考之后挂载新
Node节点部分，改完配置文件，将服务加入systemctl中（systemctl enable
docker.service），然后启动即可（systemctl start docker.service ）。

**4.2  搭建kubelet**

创建kubelet访问apiserver的证书和私钥，kubelet-csr.json（此步骤在10.126.88.3机器的/root/cfssl-online/kubernetes/node211目录下进行）：

{

"CN": "system:node:node211",

"hosts": [

],

"key": {

"algo": "rsa",

"size": 2048

},

"names": [

{

"C": "CN",

"ST": "BeiJing",

"L": "BeiJing",

"O": "system:nodes",

"OU": "cloudnative"

}

]

}

注意：

-   CN为用户名，使用system:node:\<node-name\>

-   O为用户组，Kubernetes RBAC定义了ClusterRoleBinding将Group
    system:nodes和ClusterRole system:node关联

之后生成证书和私钥：

cfssl gencert -ca=../ca.pem -ca-key=../ca-key.pem -config=../ca-config.json
-profile=wcs kubelet-csr.json \| cfssljson -bare kubelet

将生成的 kubelet.pem和 kubelet-key.pem文件拷贝到10.126.82.211机器上的
/etc/kubernetes/pki目录下，将88.3上使用的ca.pem也拷贝到此目录下。

然后在211机器上的/etc/kubernetes/创建kubelet.sh脚本，kubelet.sh脚本内容如下：

cd /etc/kubernetes

export KUBE_APISERVER="https://10.126.73.153:8123"

\# set-cluster

kubectl config set-cluster kubernetes \\

\--certificate-authority=/etc/kubernetes/pki/ca.pem \\

\--embed-certs=true \\

\--server=\${KUBE_APISERVER} \\

\--kubeconfig=kubelet.conf

\# set-credentials

kubectl config set-credentials system:node:node211 \\

\--client-certificate=/etc/kubernetes/pki/kubelet.pem \\

\--embed-certs=true \\

\--client-key=/etc/kubernetes/pki/kubelet-key.pem \\

\--kubeconfig=kubelet.conf

\# set-context

kubectl config set-context system:node:node211\@kubernetes \\

\--cluster=kubernetes \\

\--user=system:node:node211 \\

\--kubeconfig=kubelet.conf

\# set default context

kubectl config use-context system:node:node211\@kubernetes
--kubeconfig=kubelet.conf

运行此脚本。

**注意：**

1.8版本需要手动增加 节点名，
参见<https://blog.csdn.net/qq_21816375/article/details/80032934>

具体命令为：kubectl create clusterrolebinding kubelet-node-clusterbinding
--clusterrole=system:node --user=system:node:node211 \@hxx 其中node211为节点名

接下来修改/usr/lib/systemd/system/kubelet.service文件。

>   [Unit]

>   Description=Kubernetes Kubelet Server

>   Documentation=https://github.com/GoogleCloudPlatform/kubernetes

>   After=docker.service

>   Requires=docker.service

>   [Service]

>   WorkingDirectory=/opt/wcs/kubernetes/kubelet

>   EnvironmentFile=-/etc/kubernetes/config

>   EnvironmentFile=-/etc/kubernetes/kubelet

>   ExecStart=/usr/bin/kubelet \\

>   \--root-dir=/opt/wcs/kubernetes/kubelet \\

>   \--logtostderr=false \\

>   \--v=2 \\

>   \--log-dir=/opt/wcs/kubernetes/log/kubelet \\

>   \--address=10.126.82.211 \\

>   \--cluster-dns=10.96.0.10 \\

>   \--cluster-domain=cluster.local \\

>   \--kubeconfig=/etc/kubernetes/kubelet.conf \\

>   \--pod-manifest-path=/etc/kubernetes/manifests \\

>   \--allow-privileged=true \\

>   \--authorization-mode=Webhook \\

>   \--client-ca-file=/etc/kubernetes/pki/ca.pem \\

>   \--eviction-pressure-transition-period=8m0s \\

>   \--eviction-hard=memory.available\<1.5Gi \\

>   \--eviction-soft=memory.available\<2Gi \\

>   \--eviction-soft-grace-period=memory.available=2m0s \\

>   \--kube-reserved=cpu=100m,memory=500M \\

>   \--system-reserved=cpu=100m,memory=500M \\

>   \--max-pods=255 \\

>   \--cgroup-driver=systemd \\

>   \--fail-swap-on=false

>   Restart=on-failure

>   [Install]

>   WantedBy=multi-user.target

>   其中
>   --pod-manifest-path=/etc/kubernetes/manifests指定了静态Pod定义的目录。可以提前创建好这个mkdir
>   -p /etc/kubernetes/manifests。

之后启动 kubelet ：

systemctl daemon-reload

systemctl enable kubelet

systemctl start kubelet

systemctl status kubelet -l

**4.3  搭建kube-proxy**

创建kube-proxy访问apiserver的证书和私钥（kube-proxy的证书和私钥不区分node节点，所以只需要制作一份即可每个节点都使用同一份文件，在10.126.88.3机器的/root/cfssl-online/kubernetes目录下创建
kube-proxy-csr.json文件）：

{

"CN": "system:kube-proxy",

"hosts": [

],

"key": {

"algo": "rsa",

"size": 2048

},

"names": [

{

"C": "CN",

"ST": "BeiJing",

"L": "BeiJing",

"O": "system:kube-proxy",

"OU": "cloudnative"

}

]

}

其中 CN 指定该证书的 User 为system:kube-proxy。Kubernetes RBAC Kubernetes RBAC
定义了ClusterRoleBinding 将 system:kube-proxy 用户与 system:node-proxier
角色绑定。system:node-proxier 具有 kube-proxy 组件访问 ApiServer 的相关权限。

接下来生成证书：

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=wcs
kube-proxy-csr.json \| cfssljson -bare kube-proxy

将生成的kube-proxy-key.pem和kube-proxy.pem文件拷贝到10.126.82.211机器上的/etc/kubernetes/pki目录下。

然后在10.126.82.211机器上/etc/kubernetes/pki目录下编写生成kubeconfig文件的脚本kube-proxy.sh,内容如下：

cd /etc/kubernetes

export KUBE_APISERVER="https://10.126.73.153:8123"

\# set-cluster

kubectl config set-cluster kubernetes \\

\--certificate-authority=/etc/kubernetes/pki/ca.pem \\

\--embed-certs=true \\

\--server=\${KUBE_APISERVER} \\

\--kubeconfig=kube-proxy.conf

\# set-credentials

kubectl config set-credentials system:kube-proxy \\

\--client-certificate=/etc/kubernetes/pki/kube-proxy.pem \\

\--embed-certs=true \\

\--client-key=/etc/kubernetes/pki/kube-proxy-key.pem \\

\--kubeconfig=kube-proxy.conf

\# set-context

kubectl config set-context system:kube-proxy\@kubernetes \\

\--cluster=kubernetes \\

\--user=system:kube-proxy \\

\--kubeconfig=kube-proxy.conf

\# set default context

kubectl config use-context system:kube-proxy\@kubernetes
--kubeconfig=kube-proxy.conf

运行脚本。

之后修改文件/usr/lib/systemd/system/kube-proxy.service：

[Unit]

Description=Kubernetes Kube-Proxy Server

Documentation=https://github.com/GoogleCloudPlatform/kubernetes

After=network.target

[Service]

EnvironmentFile=-/etc/kubernetes/config

EnvironmentFile=-/etc/kubernetes/proxy

ExecStart=/usr/bin/kube-proxy \\

\--logtostderr=false \\

\--v=2 \\

\--log-dir=/opt/wcs/kubernetes/log/kube-proxy \\

\--bind-address=10.126.82.211 \\

\--kubeconfig=/etc/kubernetes/kube-proxy.conf \\

\--cluster-cidr=10.244.0.0/16

Restart=on-failure

LimitNOFILE=65536

然后启动kube-proxy：

systemctl daemon-reload

systemctl enable kube-proxy

systemctl start kube-proxy

systemctl status -l kube-proxy

**4.4  安装flannel插件**

flannel以DaemonSet的形式运行在 Kubernetes
集群中。由于我们的etcd集群启用了TLS认证，为了从flannel容器中能访问etcd，我们先把etcd
的TLS 证书信息保存到Kubernetes的Secret中。

在10.126.88.3机器上运行一下命令：

kubectl create secret generic etcd-tls-secret --from-file=/etc/etcd/ssl/etcd.pem
--from-file=/etc/etcd/ssl/etcd-key.pem --from-file=/etc/etcd/ssl/ca.pem -n
kube-system

运行之后查看kubectl describe secret etcd-tls-secret -n kube-system

确认创建secret成功之后，在10.126.82.211机器上创建flannel安装目录：

mkdir -p \~/k8s/flannel

进入该目录，下载相应的yaml文件：

>   wget
>   <https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml>

>   wget
>   <https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml>

>   对kube-flannel.yml文件进行以下修改：

>   \---

>   apiVersion: v1

>   kind: ServiceAccount

>   metadata:

>   name: flannel

>   namespace: kube-system

>   \---

>   kind: ConfigMap

>   apiVersion: v1

>   metadata:

>   name: kube-flannel-cfg

>   namespace: kube-system

>   labels:

>   tier: node

>   app: flannel

>   data:

>   cni-conf.json: \|

>   {

>   "name": "cbr0",

>   "type": "flannel",

>   "delegate": {

>   "isDefaultGateway": true

>   }

>   }

>   net-conf.json: \|

>   {

>   "Network": "10.244.6.1/24",

>   "Backend": {

>   "Type": "vxlan"

>   }

>   }

>   \---

>   apiVersion: extensions/v1beta1

>   kind: DaemonSet

>   metadata:

>   name: kube-flannel-ds

>   namespace: kube-system

>   labels:

>   tier: node

>   app: flannel

>   spec:

>   template:

>   metadata:

>   labels:

>   tier: node

>   app: flannel

>   spec:

>   hostNetwork: true

>   nodeSelector:

>   beta.kubernetes.io/arch: amd64

>   tolerations:

>   \- key: node-role.kubernetes.io/master

>   operator: Exists

>   effect: NoSchedule

>   serviceAccountName: flannel

>   containers:

>   \- name: kube-flannel

>   image: quay.io/coreos/flannel:v0.9.1-amd64

>   \# command: [ "/opt/bin/flanneld",

>   \# "--ip-masq",

>   \# "--kube-subnet-mgr",

>   \#
>   "--etcd-endpoints=https://10.136.196.80:6544,https://10.136.196.82:6544,https://10.136.196.84:6544",

>   \# "--etcd-cafile=/etc/etcd/ssl/ca.pem",

>   \# "--etcd-certfile=/etc/etcd/ssl/etcd.pem",

>   \# "--etcd-keyfile=/etcd/etcd/ssl/etcd-key.pem",

>   \# "--iface=eth1" ]

>   command: [ "/bin/sh","-c",

>   "/opt/bin/flanneld --ip-masq --kube-subnet-mgr
>   --etcd-endpoints=https://10.126.88.3:6544,https://10.126.88.5:6544,https://10.126.88.6:6544
>   --etcd-cafile=/etc/etcd/ssl/ca.pem --etcd-certfile=/etc/etcd/ssl/etcd.pem
>   --etcd-keyfile=/etcd/etcd/ssl/etcd-key.pem --iface=\`cat
>   /opt/network/network.conf\`" ]

>   securityContext:

>   privileged: true

>   env:

>   \- name: POD_NAME

>   valueFrom:

>   fieldRef:

>   fieldPath: metadata.name

>   \- name: POD_NAMESPACE

>   valueFrom:

>   fieldRef:

>   fieldPath: metadata.namespace

>   volumeMounts:

>   \- name: etcd-tls-secret

>   readOnly: true

>   mountPath: /etc/etcd/ssl/

>   \- name: run

>   mountPath: /run

>   \- name: flannel-cfg

>   mountPath: /etc/kube-flannel/

>   \- name: network-conf

>   mountPath: /opt/network/

>   \- name: install-cni

>   image:
>   wcs.online.registry.58corp.com:8000/wcs-kube-system/flannel:v0.9.1-amd64

>   command: [ "/bin/sh", "-c", "set -e -x; cp -f
>   /etc/kube-flannel/cni-conf.json /etc/cni/net.d/10-flannel.conf; while true;
>   do sleep 3600; done" ]

>   volumeMounts:

>   \- name: cni

>   mountPath: /etc/cni/net.d

>   \- name: flannel-cfg

>   mountPath: /etc/kube-flannel/

>   volumes:

>   \- name: etcd-tls-secret

>   secret:

>   secretName: etcd-tls-secret

>   \- name: run

>   hostPath:

>   path: /run

>   \- name: cni

>   hostPath:

>   path: /etc/cni/net.d

>   \- name: flannel-cfg

>   configMap:

>   name: kube-flannel-cfg

>   \- name: network-conf

>   hostPath:

>   path: /etc/kubernetes/flannel

flanneld的启动参数中加入以下参数

-   \-etcd-endpoints配置etcd集群的访问地址

-   \-etcd-cafile配置etcd的CA证书,/etc/etcd/ssl/ca.pem从etcd-tls-secret这个Secret挂载

-   \--etcd-certfile配置etcd的公钥证书,/etc/etcd/ssl/etcd.pem从etcd-tls-secret这个Secret挂载

-   \--etcd-keyfile配置etcd的私钥,/etc/etcd/ssl/etcd-key.pem从etcd-tls-secret这个Secret挂载

-   \--iface当Node节点有多个网卡时用于指明具体的网卡名称

>   部署flannel：

kubectl create -f kube-flannel-rbac.yml

kubectl create -f kube-flannel.yml

**4.5  部署kube-dns**

Kubernetes支持kube-dns以Cluster Add-On的形式运行。

Kubernetes会在集群中调度一个DNS的Pod和Service。同样在10.126.82.211机器上：

mkdir -p \~/k8s/kube-dns

cd \~/k8s/kube-dns

wget
https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/kubedns-cm.yaml

wget
https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/kubedns-sa.yaml

wget
https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/kubedns-svc.yaml.base

wget
https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/kubedns-controller.yaml.base

wget
https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/dns/transforms2sed.sed

查看transforms2sed.sed，将\$DNS_SERVER_IP替换成10.96.0.10，将DNS_DOMAIN替换成cluster.local。注意\$DNS_SERVER_IP要和kubelet设置的--cluster-dns参数一致。

然后执行：

sed -f transforms2sed.sed kubedns-svc.yaml.base \> kubedns-svc.yaml

sed -f transforms2sed.sed kubedns-controller.yaml.base \>kubedns-controller.yaml

之后创建kube-dns：

kubectl create -f kubedns-cm.yaml

kubectl create -f kubedns-sa.yaml

kubectl create -f kubedns-svc.yaml

kubectl create -f kubedns-controller.yaml

kube-dns 是 Kubernetes 实现服务发的重要组件之一，默认情况下只会创建一个DNS
Pod，在生产环境中我们可能需要对 kube-dns 进行扩容。

kubectl kubectl -- namespace=kube namespace=kube namespace=kube
namespace=kubenamespace=kube-system scale deployment kube-dns
--replicas=\<NUM_YOU_WANT\>

**4.6  部署dashboard插件**

在10.126.82.211机器上：

mkdir -p \~/k8s/dashboard

cd \~/k8s/dashboard

wget
<https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml>

kubectl create -f kubernetes-dashboard.yaml

**4.7  部署heapster插件**

在 10.126.82.211 机器 上：

mkdir -p \~/k8s/heapster

cd \~/k8s/heapster

wget
https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/grafana.yaml

wget
https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/rbac/heapster-rbac.yaml

wget
https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/heapster.yaml

wget
https://raw.githubusercontent.com/kubernetes/heapster/master/deploy/kube-config/influxdb/influxdb.yaml

kubectl create -f ./

最后确认所有pod都在running状态。

**4.8  云搜二期升级的操作说明**

**4.8.1  搭建Flannel**

本次搭建的flannel是0.9.1版本，本地镜像库没有该版本，因此需要拉取相应的版本。

**\#首先拉取flannel的镜像v0.9.1**

docker pull quay.io/coreos/flannel:v0.9.1-amd64

**\#修改tag：**

docker tag quay.io/coreos/flannel:v0.9.1-amd64
10.136.196.207:8000/wcs-kube-system/flannel:v0.9.1-amd64

**\#上传到本地仓库：**

docker push 10.136.196.207:8000/wcs-kube-system/flannel:v0.9.1-amd64

**4.8.2  搭建node**

1.  搭建docker

2.  搭建kubelet

>   1.8.13版本的kubernetes配置发生了变化：

>   增加： --fail-swap-on=false

>   否则可能导致在开启 swap 分区的机器上无法启动 kubelet，详细可参考
>   CHANGELOG(before-upgrading 第一条)

>   移除：

>   \--api-servers=https://10.136.96.126:8123 \\ 这个地址会在kubelet.conf中获取

>   \--require-kubeconfig 选项，已经过时废弃

>   验证方法：

>   查看kubernetes的修改日志（CHANGELOG），手动执行验证。

>   kubectl get nodes

3.  搭建kubeproxy

**5.  root账户搭建，work 账户运维**

**5.1  kubectl的work权限**

需要拷贝root下的\~/.kube/ ， 到work对应home 目录

**5.2  Docker的work权限**

docker 使用work 来执行运维命令，如docker ps
先使用root账户，创建docker组，然后将work 加入docker组中，重启docker，命令如下

sudo groupadd docker

gpasswd -a work docker

systemctl restart docker

使用work账户重新登录机器，即可执行命令

**5.3  ETCD的work权限**

**a：添加访问权限：**

对etcd 可以添加 用户名 + 密码，
详情可参见：http://ju.outofmemory.cn/entry/277465

则，对应的shell中的api 需要加上参数： -u user:password

b： etcdctl 需要对/etc/etcd/ssl添加相应的work权限

chown -R work /etc/etcd/ssl/\*
