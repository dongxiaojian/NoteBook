Docker将镜像文件运行起来后， 产生的对象就是`容器`，。 `容器`相当于是镜像运行起来的一个实例。`容器`具备一定的生命周期。可以用 `docker  ps`命令查看运行的容器。`容器`  就是一个镜像，运行起来之后产生的进程。 等后续的笔记会解释镜像与容器之间的关系，这节按住不表，稍后别的章节分析。
![docker生命周期](https://upload-images.jianshu.io/upload_images/11227136-b6e17117c4dca943.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###  0.容器生命周期(常用命令以及参数)
###### 0.1 容器创建(利用镜像创建出一个Created状态的待启动容器)
    docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
      OPTIONS:
             -t     --tty                        分配一个虚拟终端
             -i     --interactive                即使没有连接，也要保持STDIN打开
                     --name                      为常见的容器起名，没指定会随机产生一个名称
      COMMAND：
           命令参数， 表示容器启动后，需要在容器中执行的命令，linux命令或者其他的命令。
       ARG：
          COMMAND 命令的参数。
        


###### 0.2 容器启动(将一个或多个处于Created状态(创建状态)或exited(关闭状态)的容器启动起来)
      
    docker start [OPTIONS] CONTAINER [CONTAINER...]
      OPTIONS：
          -a  --attach     将当前shell的 STDOUT/STDERR 连接到容器上
          -i   --interactive   将当前的shell的STDIN连接到容器上

   ![创建启动](https://upload-images.jianshu.io/upload_images/11227136-68c2cdf602fac133.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  ###### 0.3 容器运行(将创建和启动合二为一)
         docker run   利用镜像创建并启动一个容器
           docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
          OPTIONS:
                  -t  --tty 分配一个虚拟终端
                  -i   --interactive  即使没有连接， 也要保持STDIN打开
                        --name    为容器命名，如果没有指定将会随机产生一个名称
                  -d  --detach    再后台运行容器并打印容器id
                        --rm   当容器退出运行后，自动删除容器

      COMMAND/ARG：
            COMMAND表示容器启动后， 需要在容器中执行的命令，ARG表示执行COMMAND时需要提供的一些参数。

![docker run](https://upload-images.jianshu.io/upload_images/11227136-b615ee64c881d5dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以发现：
  * docker run 相当于 docker create + docker start -a    前台模式
 * docker run -d 相当于 docker create+ docker  start  后台模式


###### 0.3 容器暂停与取消暂停
  
    docker pause    CONTAINER      暂停一个或者多个处于运行状态中的容器
    docker uppause  CONTAINER  取消一个或者多个处于暂停状态的容器，恢复运行

   ![暂停与取消暂停](https://upload-images.jianshu.io/upload_images/11227136-f89c02515f262937.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 0.4 容器关闭与终止
    

###### 0.4.1 容器关闭
    docker stop  关闭一个或者多个处于暂停状态或者运行状态的容器
     命令格式：
       docker stop [OPTIONS] CONTAINER [CONTAINER...]
      OPTIONS：
            -t, --time int   关闭前等待的时间，单位秒， 默认为10
            
###### 0.4.2 容器终止
    docker kill 强制并立即关闭一个或者多个处于暂停状态或者运行状体的容器
    命令格式：
        docker kill [OPTIONS] CONTAINER [CONTAINER...]
    OPTIONS：
        -s, --signal string   指定发送给容器的关闭信号(默认"kill")

![stop and  kill](https://upload-images.jianshu.io/upload_images/11227136-bf46418fe1ba2de6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

stop 与 kill 的区别：
       docker stop 会先发出SIGTERM信号给进程， 告诉进程即将关闭。 在指定-t的时间过了之后， 会立即发出SIGKILL信号， 直接关闭容器。
      docker kill 直接发送SIGKILL信号关闭容器， 但可以通过-s参数修改发出的信号。
    简单的讲：docker stop发出的SIGTERM信号可以被阻塞或者终止，就是说容器最终不一定被终止；docker kill 立刻发生，无法撤销；


###### 0.5 容器重启
    docker restart：重启一个或者多个处于运行状态、暂停状态、关闭状态或者新建状态的容器。
    docker restart [OPTIONS] CONTAINER [CONTAINER...]
    OPTIONS：
        -t, --time int 重启前，等待的时间， 单位秒(默认10s)，实际上是关闭前的等待


###### 0.6 容器删除
      docker rm 删除一个或者多个容器
      命令格式：
        docker rm [OPTIONS] CONTAINER [CONTAINER...] 
    OPTIONS：
        -f  --force   强制移除正在运行的容器(使用SIGKILL)
        -l  --link  删除指定的链接
        -v  --volumes  移除相关联的数据卷



### 1. 容器信息查看
###### 1.1 容器详细信息查看
      docker inspect  : 查看本地一个或者多个容器的详细信息
      命令格式：
        docker inspect [OPTIONS] NAME|ID [NAME|ID...]
      OPTIONS：
          -f  --format string  使用go语言进行格式化输出
          -s  --size  显示容器文件的总大小
                 --type string 为制定类型返回json

###### 1.2 容器日志信息
        docker logs 查看容器的日志信息
        命令格式：
            docker logs [OPTIONS] CONTAINER
        OPTIONS：
                --details   显示提供给日志的额外详细信息
              -f  --follow   动态跟踪显示日志
                   --since string  值显示某时间节点之后的
                   --tail string   显示倒数的行数(默认全部)
              -t --timestamps  只显示时间戳
                   --until string   只显示某时间节点之前的
###### 1.3 容器重命名
      docker rename  修改容器名称
       命令格式：
            docker rename CONTAINER NEW_NAME
### 2.容器登录(容器链接)
###### 2.1 attach
     docker attach 将本地标准输入，输出和错误流附加到正在运行的容器
    命令格式：
        docker attach [OPTIONS] CONTAINER
    OPTIONS：
        --detach-keys	覆盖用于分离容器的键序列
        --no-stdin		不要附上STDIN
        --sig-proxy		代理收到所有进程的信号，默认为true

###### 2.2 exec
    docker exec  在正在运行的容器中运行命令  
    命令格式：
      docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
    OPTIONS：
      -d   --detach                分离模式：在后台运行命令
           --detach-keys           覆盖用于分离容器的键序列
      -e   --env                   设置环境变量 
      -i   --interactive           即使没有连接，也要保持STDIN打开 
           --privileged            为命令提供扩展权限
      -t   --tty                   分配伪TTY
      -u   --user                  用户名或UID（格式：<name | uid> [：<group | gid>]）
      -w   --workdir               容器内的工作目录

###### 2.3 ssh
    之后的绑定网卡之后， 可以在容器之内运行ssh服务，然后进行ssh进行链接, 但是我看资料， 并不不建议这么做。

###### 2.4 nsenter
  使用nsenter工具进入docker
  ###### 2.4.1 nsenter命令直接使用：
      1. docker inspect CONTAINER -f {{.State.Pid}}    获取制定容器的进程
      2. nsenter --target 容器进程id --mount --uts --ipc --net --pid     输入该命令便进入到容器中
      # 命令解释：
            --mount参数是进去到mount namespace中
            --uts参数是进入到uts namespace中
             --ipc参数是进入到System V IPC namaspace中
            --net参数是进入到network namespace中
            --pid参数是进入到pid namespace中
            --user参数是进入到user namespace中

  ###### 2.4.2 nsenter之sh文件封装：
      这个网上有很多封装好的功能多的， 直接拿来用就ok了。
      我这里提供一个别的封装好的：
      链接: https://pan.baidu.com/s/1JKhakAAB1wWOEp_D_-Z5yA 提取码: 4yix 
      将文件的内容放到.bashrc文件中， 然后执行source .bashrc
      docker-pid  CONTAINER   获取pid
      docker-enter  获取的pid    进入到指定的container
     
  注： 这几种链接docker容器的方式都可行， 但是各有各的弊端， 因为我也是初学，其中利弊也不甚了解，不敢妄言，这详细的东西等有深刻的体会之后再来补充。


# *注：这次笔记的命令都比较多，只是列举了可能比较常用的，具体的在用到时可以参考docker的文档*
#   `如有问题欢迎留言，共同讨论！`





    

      














