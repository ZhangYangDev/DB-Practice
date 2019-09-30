## MySQL PXC 集群

### 拉取PXC镜像
```shell
[root@dk ~]# percona/percona-xtradb-cluster:5.7    # 拉取镜像
[root@dk ~]# docker tag percona/percona-xtradb-cluster pxc # 为镜像改名
[root@dk ~]# docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
pxc                                  latest              aef0b3032083        2 days ago          827MB
percona/percona-xtradb-cluster:5.7      latest              aef0b3032083        2 days ago          827MB
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

1. pmaster

```shell
docker run -di \
--name=pn1 \
--net=sqlnet \
-p 9000:3306 \
--privileged \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=cluster1 \
-e XTRABACKUP_PASSWORD=123456  \
percona/percona-xtradb-cluster:5.7 
```


2. pnode2
```shell

docker run -di \
--name=pn2 \
--net=sqlnet \
-p 9001:3306 \
--privileged \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=cluster1 \
-e XTRABACKUP_PASSWORD=123456 \
-e CLUSTER_JOIN=pn1 \
percona/percona-xtradb-cluster:5.7 

```


3. node2
```shell
docker run -di \
--name=pn3 \
--net=sqlnet \
-p 9002:3306 \
--privileged \
-e MYSQL_ROOT_PASSWORD=root \
-e CLUSTER_NAME=cluster1 \
-e XTRABACKUP_PASSWORD=123456 \
-e CLUSTER_JOIN=pn1 \
percona/percona-xtradb-cluster:5.7 
```



参数说明：

-e MYSQL_ROOT_PASSWORD : 创建的MySQL实例的密码 
-e CLUSTER_NAME=PXC \  : 创建出来的集群的名称
-e XTRABACKUP_PASSWORD=root : 创建好的集群直接同步数据的密码
-e CLUSTER_JOIN=node1  : 加入到哪个节点中



### docker-compose 

```yml
version: '3'
services:
  pmaster:
    image: percona/percona-xtradb-cluster:5.7
    container_name: pmaster
    ports:
      - 3306:3306
    networks:
      - pxcnet
    volumes:
      - pxcmaster:/var/lib/mysql
    environment:
      - "CLUSTER_NAME=dbcluster"
      - "XTRABACKUP_PASSWORD=root"
      - "MYSQL_ROOT_PASSWORD=root"
      - "TZ=Asia/Shanghai"
  pnode1:
    image: percona/percona-xtradb-cluster:5.7
    container_name: pnode1
    ports:
      - 3307:3306
    networks:
      - pxcnet
    depends_on:
      - pmaster
    volumes:
      - pxcnode1:/var/lib/mysql
    environment:
      - "CLUSTER_NAME=dbcluster"
      - "XTRABACKUP_PASSWORD=root"
      - "CLUSTER_JOIN=pmaster"
      - "TZ=Asia/Shanghai"
  pnode2: 
    image: percona/percona-xtradb-cluster:5.7
    container_name: pnode2
    ports:
      - 3308:3306
    networks:
      - pxcnet
    volumes:
      - pxcnode2:/var/lib/mysql
    depends_on:
      - pmaster
    environment:
      - "CLUSTER_NAME=dbcluster"
      - "XTRABACKUP_PASSWORD=root"
      - "CLUSTER_JOIN=pmaster"
      - "TZ=Asia/Shanghai"

networks:
  pxcnet:
    driver: bridge

volumes:
  pxcmaster:
  pxcnode1:
  pxcnode2:

```



## 验证集群状态

```sql
 show status like '%wsrep%';
```




## 参考文章


https://blog.csdn.net/weixin_34004576/article/details/94305979  
