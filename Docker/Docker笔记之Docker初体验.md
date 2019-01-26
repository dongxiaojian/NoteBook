
`听闻Docker好用，方便部署，抽个时间，系统学习之！`
一般用的版本是Docker-CE版本即Docker社区版，发布的版本为Stable(按季度发布的稳定版)和Edge(按月发布的测试版)。
##0.Docker-CE安装
###### 0.1 检查CentOS版本是否支持Docker
    Docker 要求 CentOS 系统的内核版本高于 3.10 ， 安装前先确认下系统是否支持Docker。
    
    uname -r   # 来检查当前系统的版本号

###### 0.2 确保yum为最新
    sudo yum update
 
###### 0.3 安装一些必要工具
    #  yum-util 提供yum-config-manager功能下边要用到
    sudo yum install -y yum-utils device-mapper-persistent-data lvm2

###### 0.4 添加软件源信息
    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
###### 0.5 查看可以安装的版本信息
    yum list docker-ce --showduplicates
    
###### 0.6 安装Docker
    1. 默认版本
    sudo yum install docker-ce # 默认装最新的(我的版本是18.09.0)
    
    2.指定版本
    sudo yum install docker-ce-18.06.0.ce  # 指定版本号安装(这个我没有试)

###### 0.7 开启Docker服务
    sudo service docker start
    
###### 0.8 检查Docker服务是否开启
    sudo docker version
    

### 1.加速器配置
由于docker的镜像源都在国外，由于不可描述的原因，建议配置加速器，以提升幸福感。

###### 1.1 获取自己的加速器
[阿里云加速器获取](https://dev.aliyun.com/search.html)

![阿里云加速器获取1](https://upload-images.jianshu.io/upload_images/11227136-a8011f08f58335e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![阿里云加速器获取2](https://upload-images.jianshu.io/upload_images/11227136-c9eca43a73b4ce69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按着阿里云上上边的提示修改配置文件即可！

到此恭喜你Docker安装配置成功了！下边是Docker镜像的一些操作。

### 2.Docker镜像管理
###### 2.1 镜像搜索(搜索 Docker Hub(镜像仓库)上的镜像)
    
    docker search
    
        -f  --filter  		    根据提供的条件过滤输出
            --format		    使用Go模板进行漂亮的打印搜索
            --limit	int	        最大搜索结果数 默认是25个
            --no-trunc		    不要截断输出，即显示全部内容

![docker search](https://upload-images.jianshu.io/upload_images/11227136-64ed7d758866d25c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###### 注：其中official 表示官方镜像  至于 Automated请看下边的描述

    The Official and Automated build statuses. Official repositories are built and maintained by the Stackbrew project, and Automated repositories are Automated Builds that allow you to validate the source and content of an image.



###### 2.2 查看本地镜像
    
    docker images
        -a      --all 列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）
    
                --digests :显示镜像的摘要信息
            
         -f     --filter filter:显示满足条件的镜像
            
                --format str :指定返回值的模板文件
            
                --no-trunc   显示完整的镜像信息
            
         -q     --quiet     只显示镜像ID。

![docker image](https://upload-images.jianshu.io/upload_images/11227136-fb58d33d935a6322.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


    
###### 2.3 镜像下载（从镜像仓库中下载镜像）
    
        docker pull
             -a, --all-tags  下载所有符合指定tag的镜像
![docker pull python](https://upload-images.jianshu.io/upload_images/11227136-afda901866b8d4e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





             
             
###### 2.4 删除镜像
    docker image rm  <==> docker rmi
    
        -f  --force  强制删除

  ![docker rmi](https://upload-images.jianshu.io/upload_images/11227136-9cd262e5a71f13d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





###### 2.5 镜像备份
    docker save
          -o, --output string  指定写入文件的路径字符串
          
###### 2.6 镜像备份倒入
    docker load
      -i, --input string   指定倒入文件的路径
        -q, --quiet         不打印倒入信息


![保存 导入](https://upload-images.jianshu.io/upload_images/11227136-65233a44dc6b13b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 注:保存的时候如果按image id 保存，当安装的时候，会有意外惊喜





        
        
###### 2.7 镜像详细信息
    docker image inspect <==> docker inspect
        -f -- format string  用go语言的format格式输出
        
###### 2.8 镜像历史信息
    docker history
           --format string   依旧是go语言漂亮输出
      -H, --human           将创建时间、大小进行优化打印 (default true)
          --no-trunc        显示完整信息
      -q, --quiet           只打印镜像ID
        

######  注：因为是初学， 后边这两个我也不太明白，先留着， 学完再补！


[参考：官方安装教程](https://docs.docker.com/install/linux/docker-ce/centos/)


#   `如有问题欢迎留言，共同讨论！`











