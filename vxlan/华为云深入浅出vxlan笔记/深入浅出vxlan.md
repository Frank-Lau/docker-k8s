# 深入浅出vxlan

![屏幕快照 2020-07-05 下午2.26.47](/Users/xiaodongliu/Desktop/docker-k8s/vxlan/华为云深入浅出vxlan笔记/assets/屏幕快照 2020-07-05 下午2.26.47.png)



![屏幕快照 2020-07-06 上午11.31.17](/Users/xiaodongliu/Desktop/docker-k8s/vxlan/华为云深入浅出vxlan笔记/assets/屏幕快照 2020-07-06 上午11.31.17.png)



![屏幕快照 2020-07-06 上午11.32.41](/Users/xiaodongliu/Desktop/docker-k8s/vxlan/华为云深入浅出vxlan笔记/assets/屏幕快照 2020-07-06 上午11.32.41.png)



STP防环协议,防止流量环路,两个交换机之间有两条链路连在一起,将会被剪掉一条,将会带来链路本身的浪费,如果两条链路都是实际链路,使用过程中只有一条链路能正常转发流量,另一条闲置,带来链路带宽的浪费

VLAN二层网络虚拟化,把交换机做广播域隔离,呈现出若干个逻辑上的广播域,但是只能虚拟出4094个

![屏幕快照 2020-08-18 下午10.30.40](/Users/xiaodongliu/Desktop/docker-k8s/vxlan/华为云深入浅出vxlan笔记/assets/屏幕快照 2020-08-18 下午10.30.40.png)

Underlay 物理组网

overlay 让各个主机感觉像直接连在同一个交换机中,本质是一种隧道技术



![屏幕快照 2020-08-18 下午10.36.13](/Users/xiaodongliu/Desktop/docker-k8s/vxlan/华为云深入浅出vxlan笔记/assets/屏幕快照 2020-08-18 下午10.36.13.png)

SA:源

DA:目的

想要实现两端的跨主机通信,需要在原本的以太网数据帧上再次进行封装,添加VXLAN ID(24bit,即2^24的表示范围,相对VLAN技术的4094大了很多)以及UDP头部,ip和mac,通过这种格式打通隧道,先通过UDP将数据送到对端,然后拆掉UDP包头,拆掉VXLAN头部,留下来的就是原始的以太网数据帧

![屏幕快照 2020-08-18 下午11.15.19](/Users/xiaodongliu/Desktop/docker-k8s/vxlan/华为云深入浅出vxlan笔记/assets/屏幕快照 2020-08-18 下午11.15.19.png)

怎样才能在拆解udp Header时知道内层包含VXLAN封装呢?通过UDP Header的目的端口,端口为4789

![屏幕快照 2020-08-18 下午11.24.09](/Users/xiaodongliu/Desktop/docker-k8s/vxlan/华为云深入浅出vxlan笔记/assets/屏幕快照 2020-08-18 下午11.24.09.png)

