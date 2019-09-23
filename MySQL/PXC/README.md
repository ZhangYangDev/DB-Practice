## MySQL PXC 集群

### 安装PXC镜像
```shell
[root@dk ~]# docker pull percona/percona-xtradb-cluster    # 拉取镜像
[root@dk ~]# docker tag percona/percona-xtradb-cluster pxc # 为镜像改名
[root@dk ~]# docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
pxc                                  latest              aef0b3032083        2 days ago          827MB
percona/percona-xtradb-cluster       latest              aef0b3032083        2 days ago          827MB
```
### 网络数据卷配置

#### 网络配置：
```shell
[root@dk ~]# docker network create --subnet=172.18.0.0/16 pxcnet  # 创建网络
37a20b9d9e5b095a0774a468ada6af83e31e10f2853925408874fe705700d2b7
```

```shell
[root@dk ~]# docker inspect pxcnet   # 查看网络的详细配置
[
    {
        "Name": "pxcnet",
        "Id": "37a20b9d9e5b095a0774a468ada6af83e31e10f2853925408874fe705700d2b7",
        "Created": "2019-09-20T14:17:08.560743307Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

#### 数据卷配置：
分别创建五个数据卷，因为我们要搭建五个节点的PXC集群
```shell
[root@dk ~]# docker volume create vol1
vol1
[root@dk ~]# docker volume create vol2
vol2
[root@dk ~]# docker volume create vol3
vol3
[root@dk ~]# docker volume create vol4
vol4
[root@dk ~]# docker volume create vol5
vol5
```
查看创建结果:
```shell
[root@dk ~]# docker volume ls
DRIVER              VOLUME NAME
local               vol1
local               vol2
local               vol3
local               vol4
local               vol5

```
创建备份数据卷：

```shell
[root@dk ~]# docker volume create pxcbakup
```

查看创建结果：
```shell
[root@dk ~]# docker volume ls
DRIVER              VOLUME NAME
local               pxcbakup
local               vol1
local               vol2
local               vol3
local               vol4
local               vol5
```


### 创建PXC级群节点

1. node1
```shell
docker run -d -p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=PXC \
-e XTRABACKUP_PASSWORD=root \
-v vol1:/var/lib/mysql \
-v pxcbakup:/data \
--privileged \
--name=node1 \
--net=pxcnet 
pxc
```
docker run -d -p 3306:3306  \
  -e MYSQL_ROOT_PASSWORD=root \
  -e CLUSTER_NAME=PXC \
  -e XTRABACKUP_PASSWORD=root \
  -v vol1:/var/lib/mysql \
  --privileged \
  --name=node1 \
  --net=pxcnet \
  pxc


1. node2
```shell
docker run -d -p 3307:3306 \
-e MYSQL_ROOT_PASSWORD=P@ssw0rd \
-e CLUSTER_NAME=PXC \
-e XTRABACKUP_PASSWORD=P@ssw0rd \
-e CLUSTER_JOIN=node1 \
-v vol2:/var/lib/mysql \
--privileged \
--name=node2 \
--net=pxcnet \
--ip 172.18.0.3 pxc
```
3. node3
```shell
docker run -d -p 3308:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=PXC \
-e XTRABACKUP_PASSWORD=root \
-e CLUSTER_JOIN=node1 \
-v vol3:/var/lib/mysql \
--privileged \
--name=node3 \
--net=pxcnet \
--ip 172.18.0.4 pxc
```
4. node4
```shell
docker run -d -p 3309:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=PXC \
-e XTRABACKUP_PASSWORD=root \
-e CLUSTER_JOIN=node1 \
-v vol1:/var/lib/mysql \
--privileged \
--name=node4 \
--net=pxcnet \
--ip 172.18.0.5 pxc
```
5. node5
```shell
docker run -d -p 3310:3306 \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=PXC \
-e XTRABACKUP_PASSWORD=root \
-e CLUSTER_JOIN=node1 \
-v vol1:/var/lib/mysql \
--privileged \
--name=node5 \
--net=pxcnet \
--ip 172.18.0.6 pxc

```



docker run -d -p 3306:3306  \
  -e MYSQL_ROOT_PASSWORD=root \
  -e CLUSTER_NAME=PXC \
  -v vol1:/var/lib/mysql \
  --name=node1 \
  --net=pxcnet \
  pxc



参数说明：

-e MYSQL_ROOT_PASSWORD : 创建的MySQL实例的密码 
-e CLUSTER_NAME=PXC \  : 创建出来的集群的名称
-e XTRABACKUP_PASSWORD=root : 创建好的集群直接同步数据的密码
-e CLUSTER_JOIN=node1  : 加入到哪个节点中