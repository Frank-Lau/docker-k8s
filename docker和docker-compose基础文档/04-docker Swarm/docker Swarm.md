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
     #解散集群
     docker swarm leave --force
     #加入集群的节点在manager节点执行上述指令后,也需要执行docker swarm leave指令,否则将无法加入其它的swarm集群
     ```
```
    
在另一台linux上执行如下指令
     
![屏幕快照 2020-06-09 下午2.23.42](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-09 下午2.23.42.png)
        

​```shell
#显示当前swarm集群中的节点
docker node ls
#删除结点
docker node rm
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



### 		2.5关于不同swarm节点上的service与service之间的通信

swarm包含了内置的dns服务发现的功能,当我们创建一个service的时候,如果加入了overlay网络,那么swarm将会为所有加入了overlay网络的service添加一条DNS记录,通过DNS记录就可以找到ip地址,继而通过IP地址访问service,DNS记录的并不是实际的容器IP,而是service的虚拟IP(VIP),举个简单的例子,当service进行横向扩展时,会根据swarm集群中各个节点的资源情况进行调度分配到不同的worker上,既然有多个,那我到底要访问哪一个?没法访问了,而虚拟ip(vip)则不会变,一个vip对应了多个实际容器ip,并做了负载均衡

![屏幕快照 2020-06-10 下午10.28.45](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午10.28.45.png)

![屏幕快照 2020-06-10 下午11.06.55](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午11.06.55.png)



![屏幕快照 2020-06-10 下午11.10.05](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午11.10.05.png)



![屏幕快照 2020-06-10 下午11.11.31](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午11.11.31.png)

![屏幕快照 2020-08-25 下午5.17.41](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-25 下午5.17.41.png)

![屏幕快照 2020-08-25 下午5.30.47](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-25 下午5.30.47.png)

```shell
#查看dns记录
nslookup www.imooc.com#查看慕课网地址(mac自带工具)
#创建busybox service和whoami service,加入同一网络,并将whoami scale设置为2,在busybox容器中ping whoami会拿到一个ip地址,但是这个ip地址并不是这两个服务的地址(是vip)想要查看实际的ip地址,需要执行如下命令才能看到实际ip地址
nslookup tasks.whoami
```

![屏幕快照 2020-06-12 上午12.19.46](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-12 上午12.19.46.png)



当swarm集群中有任一服务对外暴露端口时(-p 8080:8080),我们可以通过任一节点的ip+port访问到该服务,当访问到不存在该服务的节点时,通过ipvs(ipvs称之为IP[虚拟服务器](https://baike.baidu.com/item/虚拟服务器/5799459)（IP Virtual Server，简写为IPVS）。是运行在[LVS](https://baike.baidu.com/item/LVS/17738)下的提供负载平衡功能的一种技术)将请求转发到存放该服务的节点中

![屏幕快照 2020-06-10 下午11.18.45](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-10 下午11.18.45.png)



```shell
#通过下方指令可查看端口的转发规则(只要访问端口为80端口的就会转发到172.19.0.2:80上)
sudo iptables -nL -t nat
```

![屏幕快照 2020-06-12 上午12.36.21](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-12 上午12.36.21.png)

```shell
ifconfig
```

![屏幕快照 2020-06-12 上午12.42.29](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-12 上午12.42.29.png)

```shell
brctl show
```

![屏幕快照 2020-06-12 上午12.55.03](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-06-12 上午12.55.03.png)

查看加入网络的容器,发现端口转发将请求转发到了ingress-sbox

![屏幕快照 2020-08-29 上午11.30.45](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 上午11.30.45.png)



```shell
sudo ls /var/run/docker/netns#查看network namespace
```

![屏幕快照 2020-08-29 上午11.35.09](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 上午11.35.09.png)



```shell
sudo nsenter --net=/var/run/docker/netns/ingress_sbox#进入该network namespace
```



![屏幕快照 2020-08-29 上午11.41.18](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 上午11.41.18.png)



```shell
#查看实际的负载均衡,在改network namespace中执行下述指令
iptables -nL -t mangle

```



![屏幕快照 2020-08-29 下午2.02.46](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 下午2.02.46.png)





使用docker-compose version 部署wordpress

```shell
version: '3'

services:

  web:#实际名字为wordpress_web(wordpress为通过docker stack指令执行改文件时起的名字)
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-network#加入的网络(自动创建),实际名字为 wordpress_my-network
    depends_on:
      - mysql
    deploy:
      mode: replicated#可扩展服务数量,global为只有一个服务
      replicas: 3#服务数量
      restart_policy:#重启策略
        condition: on-failure
        delay: 5s#两次重启尝试之间等待的时间（默认值：0)
        max_attempts: 3#尝试重新启动容器的次数（默认值：永不放弃)
      update_config:#配置应如何更新服务
        parallelism: 1#一次更新的容器数
        delay: 10s#在更新一组容器之间等待的时间

  mysql:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-network
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager#该service将部署在manager节点

volumes:
  mysql-data:

networks:
  my-network:
    driver: overlay
```



```shell
#运行该文件,创建多个service
docker stack deploy wordpress --compose-file=docker-compose.yml
#wordpress 为名字,与yml文件中的service名字不能冲突
```



![屏幕快照 2020-08-29 下午4.50.17](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 下午4.50.17.png)



```shell
#查看stack列表
docker stack ls
#查看详情
docker stack ps wordpress
#查看stack中service列表
docker stack services wordpress
#删除
docker stack rm wordpress
```

列表

![屏幕快照 2020-08-29 下午4.54.20](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 下午4.54.20.png)

详情

![屏幕快照 2020-08-29 下午4.57.35](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 下午4.57.35.png)

service 列表

![屏幕快照 2020-08-29 下午4.59.37](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 下午4.59.37.png)

删除(会将service和创建的网络一同删除)

![屏幕快照 2020-08-29 下午5.04.08](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 下午5.04.08.png)

注意:使用docker swarm部署时yml文件中不能使用build参数

![屏幕快照 2020-08-29 下午5.06.06](/Users/xiaodongliu/Desktop/docker-k8s/docker和docker-compose基础文档/04-docker Swarm/assets/屏幕快照 2020-08-29 下午5.06.06.png)

