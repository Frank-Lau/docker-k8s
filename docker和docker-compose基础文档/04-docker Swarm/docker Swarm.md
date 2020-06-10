# docker Swarm

## 	1.docker swarm架构

​				1.集群架构,分为两种角色

   - Manager  swarm集群的核心,至少两个,既然有多个,那就涉及到状态的同步,通过内置的raft协议实现Manager数据同步

   - Worker     实际工作的节点,部署的大部分容器都运行在这里,worker节点之间数据的同步通过Gossip网络做同步

   - ![屏幕快照 2020-06-09 上午12.08.25](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 上午12.08.25.png)

   - ![屏幕快照 2020-06-09 上午12.12.21](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 上午12.12.21.png)

   - ![屏幕快照 2020-06-09 上午12.18.07](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 上午12.18.07.png)

   - **docker swarm 集群创建**

     ```shell
     #创建第一个节点(将作为manager节点)
     docker swarm init --advertise-addr=192.168.205.10(当前linux ip)
     #执行上述指令之后会返回一个token,在另外一台linux上通过这个token就可以加入该集群成为worker
     docker swarm join --token token
     #总结:通过init指令创建节点,并指定本机ip,作为manager节点,返回一个token,集群中得其他成员通过该tonken加入该集群
```
     
![屏幕快照 2020-06-09 下午2.21.58](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 下午2.21.58.png)
     
在另一台linux上执行如下指令
     
![屏幕快照 2020-06-09 下午2.23.42](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 下午2.23.42.png)
     
     

```shell
#显示当前swarm集群中的节点
docker node ls
```

![屏幕快照 2020-06-09 下午2.27.12](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 下午2.27.12.png)

## 2.service的创建维护和水平扩展

### 		2.1创建

```shell
#创建service
docker service create --name demo busybox sh -c "while true;do sleep 3600;done"
#查看service
docker service ls
#查看某个service具体信息
docker service ps demo
#删除service
docker service rm demo
```

![屏幕快照 2020-06-09 下午10.31.38](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 下午10.31.38.png)



![屏幕快照 2020-06-10 下午2.04.57](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.04.57.png)

```shell
#执行docker ps之后会看到容器的名字并不是我们创建的service的名字
docker ps
```

![屏幕快照 2020-06-10 下午2.07.25](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.07.25.png)

### 		2.2水平扩展

```shell
#与docker compose类似,之后发现 REPLICAS变成了5/5(就绪5/共计5)
docker service scale demo=5
```

![屏幕快照 2020-06-10 下午2.11.58](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.11.58.png)



![屏幕快照 2020-06-10 下午2.13.16](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.13.16.png)

```shell
#执行docker service ps demo发现是平均分配到不同的节点上的
docker service ps demo
```

![屏幕快照 2020-06-10 下午2.15.22](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.15.22.png)

### 		2.3故障恢复

```shell
#去其中一台主机上将其中一个节点干掉,发现变成了4/5,过一段时间再执行发现又变成了5/5
```

![屏幕快照 2020-06-10 下午2.18.23](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.18.23.png)

### 		2.4实战演练

部署wordpress和mysql

```shell
#1.创建用户多机容器间通信的网络overlay
docker network create -d overlay demo
#2.启动mysql service
docker service create --name mysql --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress --network demo --mount type=volume,source=mysql-data,destination=/var/lib/mysql mysql#设置数据卷挂载,本地目录mysql-data,容器目录/var/lib/mysql
#3.启动wordpress
docker service create --name wordpress -p 80:80 --env WORDPRESS_DB_PASSWORD=root --env WORDPRESS_DB_HOST=mysql --network demo wordpress

```

![屏幕快照 2020-06-10 下午2.42.18](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.42.18.png)

![屏幕快照 2020-06-10 下午2.43.20](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.43.20.png)

![屏幕快照 2020-06-10 下午2.46.52](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.46.52.png)

```shell
#执行完之后会发现wordpress被分配到了worker2,可以通过worker2的ip:80访问,也可以通过任意集群节点的ip访问
```

![屏幕快照 2020-06-10 下午2.50.29](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午2.50.29.png)



