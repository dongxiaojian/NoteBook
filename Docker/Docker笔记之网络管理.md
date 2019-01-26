Docker中的容器的网络默认与宿主机、与其他容器都是相互隔离的， 实现容器之间、容器运行的服务于外部网络之间等的连接关系时那就需要Docker的网络管理了。

### 0. 概述：
###### 0.1 Docker的网络驱动模式：


          bridge network 模式(网桥)：默认的网络模式。
          host network 模式(主机)：容器与宿主机之间的网络无隔离，即容器直接使用宿主机的网络。
          null network 模式(禁用)： 容器禁用所有的网络。
          Overlay network模式(覆盖网络)： 利用VXLAN实现的bridge模式。
          Macvlan network模式：容器具备mac地址，使其显示为网络上的物理设备

###### 0.2 基本网络命令
      命令格式：
        docker network COMMAND
    commands:
         ls          网络对像的列表
         create      创建网络新的网络对象
         connect     将正在运行的容器连接到网络
         disconnect  将正在运行的容器连接的网络去掉
         inspect     查看一个或者多个网络对象的相信信息
         prune       删除所有没有使用的网络对象
         rm          删除一个或者多个网络对象


###### 0.2.0 docker network ls
  Docker安装完后，会自动创建bridge、host、none三种网络驱动模式， 所以即使没有创建新的网络对象，docker network ls 也会显示出已有的网络对象

![docker network ls](https://upload-images.jianshu.io/upload_images/11227136-7a92586277ec80bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###### 0.2.1 创建新的网络对象
   docker network create   创建新的网络对象
![docker network create --help](https://upload-images.jianshu.io/upload_images/11227136-d5f4ce4e86b7288a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`注： 1.host和none 模式网络只能存在一个。
        2.docker自带的overlay网络创建依赖于docker swarm(集群负载均衡)服务。
        3.bridge 和Macvlan可以有多个。
`
###### 0.2.2 删除网络
    docker network rm 删除网络
    命令格式：
        docker network rm NETWORK [NETWORK...]

###### 0.2.3 查看网络详情
    docker network inspect  查看一个或者多个网络的详细信息
    命令格式：
      docker network inspec  [OPTIONS] NETWORK [NETWORK...]
    OPTIONS：
        -f, --format string   使用Go语言格式化输出
        -v, --verbose          诊断的详细输出
    
 ###### 0.2.4 使用网络或断开连接  [查看更多](https://docs.docker.com/engine/reference/commandline/network_connect/#description)
   使用网络有两种方式，第一种：在运行容器的同时(docker run)，为启动的容器指定网络模式；第二种：将制定的容器与指定的网络进行连接。
使用命令 `docker network disconnect `将容器从网络中移除，即断开连接。

      第一种：
          命令格式：
          docker run --network=<network-name>   启动容器同时，将其连接到网络
     第二种：docker network connect
          命令格式：
            docker network connect [OPTIONS] NETWORK CONTAINER
          OPTIONS：
            --alias		为容器添加网络范围的别名
            --ip		IPv4地址（例如，172.30.100.104）
            --ip6		IPv6地址（例如，2001：db8 :: 33）
            --link		添加到另一个容器的链接
            --link-local-ip		为容器添加链接本地地址

###1. Docker 网络模式简介
###### 1.1 bridge 网络模式
![bridge网络示意图](https://upload-images.jianshu.io/upload_images/11227136-cc9c6f73eb67c67b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  `docker0` 是Docker安装是默认创建的bridge网卡，bridge模式在宿主机上需要单独的bridge网卡，创建一个新的bridge网络， 便会生成一个新的bridge网卡。`veth pair`  是虚拟网络设备对，一个在容器上，一个在宿主机上，容器与容器之间、容器与宿主机这件借助虚拟网络设备对进行通信。  `container1  container2 container3`容器上的ip是创建docker0时的命令参数 ` --subnet`所定的子网网段，每个容器上的ip是唯一的。至此，结构已经基本描述清楚了。那么问题来了，到底是怎么进行通信的呢？？
答案是 `端口映射`： 在创建/运行容器时进行端口映射。`docker create/run -P `将容器内所有暴露的端口(如：3306 6379)进行映射(不暴露不映射)。 `doker create/run -p`手动指定端口号，命令格式：`[host_ip]:[host_prot]:container_prot` ,通过命令可以看得出来，是将宿主机的ip和端口号映射到容器的端口上。
![docker network create my_bridge](https://upload-images.jianshu.io/upload_images/11227136-282d6c02dbbec137.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![docker run -dti --network my_bridge2 centos](https://upload-images.jianshu.io/upload_images/11227136-f52915542e62abb2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![ docker network create --subnet 172.10.0.0:16  --ip-range 172.10.1.0:24 --gateway 172.10.0.1  my_test_bridge](https://upload-images.jianshu.io/upload_images/11227136-89fba57362a7b263.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![docker随机绑定端口](https://upload-images.jianshu.io/upload_images/11227136-ba005dd54e02ba7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Docker指定端口映射](https://upload-images.jianshu.io/upload_images/11227136-52085a4367e49239.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 1.2 host网络模式
![host网络示意图](https://upload-images.jianshu.io/upload_images/11227136-698492c3b580f9fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
容器共享主机的网络，网络无隔离。宿主机的ip就是容器的ip。外部可以通过端口直接访问容器，跟bridge相比不需要端口映射。
如果容器或服务未发布端口，则host网络模式无效。
host网络驱动程序仅适用于Linux主机，并且不支持Docker for Mac，Docker for Windows或Docker EE for Windows Server。

###### 1.3 none网络模式
容器上没有网络，也无任何网络设备。如果需要使用网络，需要用户自行安装和配置。


###### 1.4 overlay网络模式
![overlay网络模式示意图](https://upload-images.jianshu.io/upload_images/11227136-8a697ddebe7313f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
overlay网络模式又称覆盖网络模式，这个我理解的也不太透彻，我的理解是在容器c1数据封装的基础上，overlay network对数据进行第二次封装，从而达到在同网段通信的目的。进而实现容器集群的通信。具体操作过程请参看 [overlay搭建过程](https://docs.docker.com/network/network-tutorial-overlay/#use-the-default-overlay-network)

###### 1.5 macvlan网络模式
![macvlan网络模式示意图](https://upload-images.jianshu.io/upload_images/11227136-526b419aad7963cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 `macvlan`网络模式其特征就是通信会直接通过基于MAC地址进行转发。 这是宿主机相当于一个二层交换机，Docker会维护着一个MAC地址表，当宿主机网络接收到一个数据包后，直接根据MAC地址找到相应的容器，然后经数据传到相应的容器。但值得注意的是，同一宿主机的容器直接拿可以直接通过ip互通，但与宿主机直间无法直接利用ip互通。具体搭建可以参看[macvlan搭建](https://docs.docker.com/network/network-tutorial-macvlan/)

##### 参考： 传智播客docker课件，以上网络模式示意图，均来自传智播客。
#### *总注：每个网络模式都有自己的应用场景，应用最多的bridge和host。其他的我并没有了解太透彻。*
