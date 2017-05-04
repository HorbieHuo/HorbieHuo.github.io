---
title: etcd配置
date: 2017-05-04 11:04:09
tags:
---

下载和解压etcd的相关文件，略过

不输的文件夹是 /var/lib/etcd，其中 etcd 和 etcdctl 需要在 /usr/bin/ 下创建相关软连接，以便后续的service文件使用，也可以修改service文件

启动 etcd 服务的命令是 sudo systemctl start etcd
启动后检查，curl http://127.0.0.1:2379/v2/members 或者使用其他工具请求这个链接，IP地址为部署的任意机器的IP
```json
{
  "members": [
    {
      "id": "5b123c0b90e015",
      "name": "etcd01",
      "peerURLs": [
        "http://192.168.56.101:2380"
      ],
      "clientURLs": [
        "http://192.168.56.101:2379"
      ]
    },
    {
      "id": "b082d7cebafd0896",
      "name": "etcd02",
      "peerURLs": [
        "http://192.168.56.102:2380"
      ],
      "clientURLs": [
        "http://192.168.56.102:2379"
      ]
    },
    {
      "id": "ea85a2410887d0cd",
      "name": "etcd03",
      "peerURLs": [
        "http://192.168.56.103:2380"
      ],
      "clientURLs": [
        "http://192.168.56.103:2379"
      ]
    }
  ]
}
```
etcdctl 查看所有集群成员, 命令：etcdctl member list
```bash
5b123c0b90e015: name=etcd01 peerURLs=http://192.168.56.101:2380 clientURLs=http://192.168.56.101:2379 isLeader=true
b082d7cebafd0896: name=etcd02 peerURLs=http://192.168.56.102:2380 clientURLs=http://192.168.56.102:2379 isLeader=false
ea85a2410887d0cd: name=etcd03 peerURLs=http://192.168.56.103:2380 clientURLs=http://192.168.56.103:2379 isLeader=false
```

---------------------

etcd使用的配置文件内容，位置：/var/lib/etcd/etcd.conf
```bash
# node-1
ETCD_NAME=etcd01
ETCD_DATA_DIR="/var/lib/etcd/data"
ETCD_LISTEN_PEER_URLS="http://192.168.56.101:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.56.101:2379,http://127.0.0.1:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.56.101:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.101:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster1"
ETCD_INITIAL_CLUSTER="etcd01=http://192.168.56.101:2380,etcd02=http://192.168.56.102:2380,etcd03=http://192.168.56.103:2380"

# node-2
ETCD_NAME=etcd02
ETCD_DATA_DIR="/var/lib/etcd/data"
ETCD_LISTEN_PEER_URLS="http://192.168.56.102:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.56.102:2379,http://127.0.0.1:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.56.102:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.102:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster1"
ETCD_INITIAL_CLUSTER="etcd01=http://192.168.56.101:2380,etcd02=http://192.168.56.102:2380,etcd03=http://192.168.56.103:2380"

# node-3
ETCD_NAME=etcd03
ETCD_DATA_DIR="/var/lib/etcd/data"
ETCD_LISTEN_PEER_URLS="http://192.168.56.103:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.56.103:2379,http://127.0.0.1:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.56.103:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.103:2379"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster1"
ETCD_INITIAL_CLUSTER="etcd01=http://192.168.56.101:2380,etcd02=http://192.168.56.102:2380,etcd03=http://192.168.56.103:2380"
```

首先贴一个etcd使用的service文件，使用systemctl启动使用的，文件位置(ubuntu 16.04) /lib/systemd/system/etcd.service
```bash
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/var/lib/etcd/etcd.conf
User=life
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd --name=\"${ETCD_NAME}\" --data-dir=\"${ETCD_DATA_DIR}\" --listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\" --listen-peer-urls=\"${ETCD_LISTEN_PEER_URLS}\" --advertise-client-urls=\"${ETCD_ADVERTISE_CLIENT_URLS}\" --initial-cluster-token=\"${ETCD_INITIAL_CLUSTER_TOKEN}\" --initial-cluster=\"${ETCD_INITIAL_CLUSTER}\" --initial-cluster-state=\"${ETCD_INITIAL_CLUSTER_STATE}\" "
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
