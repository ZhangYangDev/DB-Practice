## MySQL

PXC全称(Percona XtraDB Cluster)

docker pull mysql/mysql-cluster

### PXC 集群方案

#### 使用Percona Server来搭建PXC集群

#### 特点

速度慢、强一致性、高价值

PXC集群采用同步复制，事物在所有集群节点要么同时提交要么不提交


### Repliaction 集群方案

速度快、弱一致性、低价值


参考：

https://blog.csdn.net/annotation_yang/article/details/80860988