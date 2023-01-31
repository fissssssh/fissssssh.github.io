---
title: "Create Redis Cluster by Docker Compose"
date: 2022-06-21T15:52:38+08:00
categories: devops
tags:
  - docker
  - docker-compose
  - redis
draft: false
---

## Create a docker-compose.yml

```yaml
version: "3.9"
services:
  redis-1:
    image: redis:7.0.2-alpine
    container_name: redis-1
    ports:
      - "6371:6371"
      - "16371:16371"
    volumes:
      - ./node-1/data:/data
      - ./node-1/conf/redis.conf:/etc/redis/redis.conf
    networks:
      redis:
        ipv4_address: 172.28.0.11
    command:
      - "redis-server"
      - "/etc/redis/redis.conf"
  redis-2:
    image: redis:7.0.2-alpine
    container_name: redis-2
    ports:
      - "6372:6372"
      - "16372:16372"
    volumes:
      - ./node-2/data:/data
      - ./node-2/conf/redis.conf:/etc/redis/redis.conf
    networks:
      redis:
        ipv4_address: 172.28.0.12
    command:
      - "redis-server"
      - "/etc/redis/redis.conf"
  redis-3:
    image: redis:7.0.2-alpine
    container_name: redis-3
    ports:
      - "6373:6373"
      - "16373:16373"
    volumes:
      - ./node-3/data:/data
      - ./node-3/conf/redis.conf:/etc/redis/redis.conf
    networks:
      redis:
        ipv4_address: 172.28.0.13
    command:
      - "redis-server"
      - "/etc/redis/redis.conf"
  redis-4:
    image: redis:7.0.2-alpine
    container_name: redis-4
    ports:
      - "6374:6374"
      - "16374:16374"
    volumes:
      - ./node-4/data:/data
      - ./node-4/conf/redis.conf:/etc/redis/redis.conf
    networks:
      redis:
        ipv4_address: 172.28.0.14
    command:
      - "redis-server"
      - "/etc/redis/redis.conf"
  redis-5:
    image: redis:7.0.2-alpine
    container_name: redis-5
    ports:
      - "6375:6375"
      - "16375:16375"
    volumes:
      - ./node-5/data:/data
      - ./node-5/conf/redis.conf:/etc/redis/redis.conf
    networks:
      redis:
        ipv4_address: 172.28.0.15
    command:
      - "redis-server"
      - "/etc/redis/redis.conf"
  redis-6:
    image: redis:7.0.2-alpine
    container_name: redis-6
    ports:
      - "6376:6376"
      - "16376:16376"
    volumes:
      - ./node-6/data:/data
      - ./node-6/conf/redis.conf:/etc/redis/redis.conf
    networks:
      redis:
        ipv4_address: 172.28.0.16
    command:
      - "redis-server"
      - "/etc/redis/redis.conf"
  redis-cluster:
    image: redis:7.0.2-alpine
    networks:
      redis:
        ipv4_address: 172.28.0.17
    depends_on:
      - redis-1
      - redis-2
      - redis-3
      - redis-4
      - redis-5
      - redis-6
    command:
      - "redis-cli"
      - "--cluster"
      - "create"
      - "172.28.0.11:6371"
      - "172.28.0.12:6372"
      - "172.28.0.13:6373"
      - "172.28.0.14:6374"
      - "172.28.0.15:6375"
      - "172.28.0.16:6376"
      - "--cluster-replicas"
      - "1"
      - "--cluster-yes"
networks:
  redis:
    ipam:
      config:
        - subnet: "172.28.0.0/16"
```

There has a container named `redis-cluster`, the container is for create cluster for redis nodes, if you dont like this, you can enter any redis node container and execute the following command

```shell
redis-cli --cluster create \
172.28.0.11:6371 \
172.28.0.12:6372 \
172.28.0.13:6373 \
172.28.0.14:6374 \
172.28.0.15:6375 \
172.28.0.16:6376 \
--cluster-replicas 1 --cluster-yes
```

## Create config files for every redis node

Run the following commands to create config files

```shell
for port in $(seq 1 6); \
do \
mkdir -p ./node-${port}/conf
touch ./node-${port}/conf/redis.conf
cat << EOF > ./node-${port}/conf/redis.conf
port 637${port}
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip <your_ip>
cluster-announce-port 637${port}
cluster-announce-bus-port 1637${port}
appendonly yes
EOF
done
```
Don't forget to provide your actual IP address instead of <your_ip> (f.i. 10.12.123.124). Otherwise you nodes wouldn't join into cluster.

You can change port range if nesscessary

```diff
for port in $(seq 1 6); \
...
-port 637${port}
+port <port_prefix>${port}
...
-cluster-announce-port 637${port}
-cluster-announce-bus-port 1637${port}
+cluster-announce-port <port_prefix>${port}
+cluster-announce-bus-port <cluster_announce_bus_port_prefix>${port}
...
done
```

It also need to change docker-compose.yml

```diff
version: "3.9"
services:
  redis-1:
...
    ports:
-      - "6371:6371"
-      - "16371:16371"
+      - "<port_prefix>1:<port_prefix>1"
+      - "<cluster_announce_bus_port_prefix>1:<cluster_announce_bus_port_prefix>1"
...
  redis-2:
...
    ports:
-      - "6372:6372"
-      - "16372:16372"
+      - "<port_prefix>:<port_prefix>"
+      - "<cluster_announce_bus_port_prefix>2:<cluster_announce_bus_port_prefix>2"
...
  redis-3:
...
    ports:
-      - "6373:6373"
-      - "16373:16373"
+      - "<port_prefix>3:<port_prefix>3"
+      - "<cluster_announce_bus_port_prefix>3:<cluster_announce_bus_port_prefix>3"
...
  redis-4:
...
    ports:
-      - "6374:6374"
-      - "16374:16374"
+      - "<port_prefix>4:<port_prefix>4"
+      - "<cluster_announce_bus_port_prefix>4:<cluster_announce_bus_port_prefix>4"
...
  redis-5:
...
    ports:
-      - "6375:6375"
-      - "16375:16375"
+      - "<port_prefix>5:<port_prefix>5"
+      - "<cluster_announce_bus_port_prefix>5:<cluster_announce_bus_port_prefix>5"
...
  redis-6:
...
    ports:
-      - "6376:6376"
-      - "16376:16376"
+      - "<port_prefix>6:<port_prefix>6"
+      - "<cluster_announce_bus_port_prefix>6:<cluster_announce_bus_port_prefix>6"
...
  redis-cluster:
...
    command:
...
-      - "172.28.0.11:6371"
-      - "172.28.0.12:6372"
-      - "172.28.0.13:6373"
-      - "172.28.0.14:6374"
-      - "172.28.0.15:6375"
-      - "172.28.0.16:6376"
+      - "172.28.0.11:<port_prefix>1"
+      - "172.28.0.12:<port_prefix>2"
+      - "172.28.0.13:<port_prefix>3"
+      - "172.28.0.14:<port_prefix>4"
+      - "172.28.0.15:<port_prefix>5"
+      - "172.28.0.16:<port_prefix>6"
...
```

## Start redis cluster

Just run command `docker-compose up -d`.
