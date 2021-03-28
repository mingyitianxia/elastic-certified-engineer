# 可用docker-compose文件
---
这些文件是为了一些同学没有办法轻易找到足够多的机器资源而准备的，专门用于单机构建ES集群用以做相关练习。

## 环境准备
1. 安装所需应用
    1. docker
    1. docker-compose
1. 下载对应版本ES、Kibana镜像
    1. `docker pull elasticsearch:7.2.1`
    1. `docker pull kibana:7.2.1`
1. 创建所需要网络
    1. `docker network create bigdata`

## 启动对应docker-compose集群
1. 例：启动基础版1ES 1Kibana节点的集群。
    * `docker-compose -f 1e1k_base_cluster.yml up -d --build`

1. 三节点（1 master 2 data）1 kibana集群
    * `docker-compose -f 1m2d1k_normal_cluster.yml up -d --build`