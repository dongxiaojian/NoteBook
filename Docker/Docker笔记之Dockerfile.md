Docker通过Dockerfile包含所有命令文本文件中读取指令来按照指令顺序构建给定图像。Docker镜像由只读层组成，这些层都是堆叠的，也就是说每一层都是前一层变化的增量。也就是说通过Dockerfile文本文件的指令，来创建每一层。
### 0. 一个简单的Dockerfile使用
 ![docker build](https://upload-images.jianshu.io/upload_images/11227136-c1571710a188376c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###### 先来说下docker build

    docker build 根据Dockerfile文件创建镜像
    命令格式：
        docker build [OPTIONS] PATH | URL | -
    Options：
      --add-host           添加自定义主机到IP映射（主机：ip）
      --build-arg          设置构建时变量 
      --cache-from         要考虑作为缓存源的图像  
      --cgroup-parent      容器的可选父cgroup  
      --compress           使用gzip压缩构建上下文  
      --cpu-period        限制CPU CFS（完全公平计划程序）期间  
      --cpu-quota        限制CPU CFS（完全公平计划程序）配额 
      -c   --cpu-shares           CPU份额（相对权重） 
      --cpuset-cpus        允许执行的CPU（0-3,0,1） 
      --cpuset-mems        允许执行的MEM（0-3,0,1）  
      --disable-content-trust        跳过图像验证  默认为true
      --file , -f        Dockerfile的名称（默认为'PATH / Dockerfile'） 
      --force-rm        始终移除中间容器 
      --iidfile        将图像ID写入文件 
      --isolation      容器隔离技术 
      --label        设置图像的元数据 
       -m  --memory  内存限制 
       --memory-swap        交换限制等于内存加交换： - 1以启用无限制交换 
       --network        在构建期间设置RUN指令的网络模式 
       --no-cache        构建映像时不要使用缓存 
        --platform        如果服务器具有多平台功能，则设置平台
       --progress        设置进度输出类型（auto，plain，tty）。使用plain显示容器输出，默认为True
       --pull        始终尝试拉出较新版本的图像 
       -q  --quiet         成功时禁止构建输出并打印图像ID 
       --rm        成功构建后删除中间容器，默认为True
       --secret         用于公开构建的秘密文件（仅当启用了BuildKit时）：id = mysecret，src = / local / secret |
       --security-opt        安全选项 
       --shm-size        / dev / shm的大小  
       --squash        新构建的图层压缩到一个新图层中  
       --ssh        SSH代理套接字或公开给构建的键（仅当启用了BuildKit时）（格式：default |<id style="box-sizing: border-box;">[=<socket style="box-sizing: border-box;">|<key style="box-sizing: border-box;">[，<key style="box-sizing: border-box;">]]）</key></key></socket></id> 
       --stream     Stream附加到服务器以协商构建上下文  
       -t   --tag   以“name：tag”格式命名和选择标记 
       --target  设置要构建的目标构建阶段。  
       --ulimit   Ulimit选项 
###### 然后说下构建镜像的流程
先从`FORM`指定的镜像(如果本地没有，会自动从Docker Hub下载)，创建一个容器，然后执行'RUN'指令后边的内容，最后执行`docker commit`命令,将容器提交为镜像，最后删除过程中创建的容器。
![过程.png](https://upload-images.jianshu.io/upload_images/11227136-b9d2ba53d62a8481.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 接着来看
![不同的情况](https://upload-images.jianshu.io/upload_images/11227136-952097e3134afb55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过图片可以发现，
* 第一次执行，跟预想的一样。
* 第二次执行`docker build`时，第二步没有执行，显示的时`using cache`，Dockerfile2文件与第一次执行时的差异是在`RUN echo "测试行"`下一行添加了`RUN echo "测试行下一行1`;
* 第三次执行`docker build`时，跟预想的一样，Dockerfile2文件与第二次执行时的差异是在`RUN echo "测试行"`上一行添加了`RUN echo "测试行上一行"`;
可以发现：`当Dockerfile文件需要增加指令时，尽量在末尾添加。如果在FROM下添加，会引起不必要的资源消耗。如果内存中存在执行过的指令，指令执行是从修改行开始的。`

### 1. Dockerfile编写命令：[查看详细](https://docs.docker.com/engine/reference/builder/#usage)
    FROM: 指定基础镜像
    RUN： 构建镜像过程中需要执行的命令。可以有多条。
    CMD：添加启动容器时需要执行的命令。多条只有最后一条生效。可以在启动容器时被覆盖和修改。
    ENTRYPOINT：同CMD，但这个一定会被执行，不会被覆盖修改。
    LABEL ：为镜像添加对应的数据。
    MLABELAINTAINER：表明镜像的作者。将被遗弃，被LABEL代替。
    EXPOSE：设置对外暴露的端口。
    ENV：设置执行命令时的环境变量，并且在构建完成后，仍然生效
    ARG：设置只在构建过程中使用的环境变量，构建完成后，将消失
    ADD：将本地文件或目录拷贝到镜像的文件系统中。能解压特定格式文件，能将URL作为要拷贝的文件
    COPY：将本地文件或目录拷贝到镜像的文件系统中。
    VOLUME：添加数据卷
    USER：指定以哪个用户的名义执行RUN, CMD 和ENTRYPOINT等命令
    WORKDIR：设置工作目录
    ONBUILD：如果制作的镜像被另一个Dockerfile使用，将在那里被执行Docekrfile命令
    STOPSIGNAL：设置容器退出时发出的关闭信号。
    HEALTHCHECK：设置容器状态检查。
    SHELL：更改执行shell命令的程序。Linux的默认shell是[“/bin/sh”, “-c”]，Windows的是[“cmd”, “/S”, “/C”]。


######  *注：Dockerfile编写还得多练！*
