Docker仓库分为公有仓库和私有仓库。
  * 公有仓库值的是Docker Hub(官方库)等开放给用户使用的仓库。
* 私有仓库指的是由用户自行搭建的存放镜像的环境。
Docker Hub一般情况下，都是从上边下拉对应的环境，其他的用的不太多。这里主要做一下私有仓库的各种笔记。
######在搭建之前必须安装docker，然后从docker hub 下拉registry，registry是一个特殊的镜像，主要用来搭建仓库。
### 0. 搭建无认证私有仓库

    # 第一步： 在仓库的服器上运行registry
    docker run -d  --restart always --name my_registry -p 5000:5000 -v /my_registry/registry:/var/lib/registry registry

    执行完这一步,相当于私有仓库搭好了，在本地浏览器上查看 ip:5000/v2/_catalog中可以查看到存放的镜像列表，现在为空:`{repositories: [ ]}`
    参数含义：
     --restart  是否跟随docker服务重启
     --name 仓库名称
     -p  端口映射，前为宿主机的端口，后边是容器的端口，容器端口不可变
     -v 挂载数据卷，防止容器删除导致上传的镜像丢失 ，/var/lib/registry路径是固定的

    # 第二步：重命名需要上传的镜像，在本地服务器上执行
      docker tag IMAGES ip:port/IMAGES_NAME   # 必须重命名为这种格式
    # 第三步：利用docker push 上传刚才重命名的镜像
      docker push ip:port/IMAGES_NAME
      # 不出意外，会抛出以下错误
        The push refers to repository [ip:5000/centos_test]
        Get https://ip:5000/v2/: http: server gave HTTP response to HTTPS client
    这个错误是因为服务器没证书，不支持https导致，这时候只需要在服务其的配置文件/etc/docker/daemon.json添加 "insecure-registries":[ip:port],然后再执行push

### 1.搭建带认证的私有仓库
    # 第一步：在服务器中，创建存放认证用户名和密码的文件
       mkdir -p /my_registry/auth 
    # 第二步：在服务器上，创建密码验证文件。
       docker run  --entrypoint htpassword registry -Bbn username password > /my_registry/auth /htpassword 
    # 第三步：重新启东仓库镜像
        docker run -d -p 5001:5000 --restart=always 
        --name docker-registry 
        -v /my_registry/registry:/var/lib/registry
        -v /my_registry/auth:/auth 
        -e "REGISTRY_AUTH=htpasswd" 
        -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm"
        -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry
    参数： -p、--restart、--name，-v不再赘述
      第二个-v 表示容器验证文件挂载路径，将宿主机的验证文件复制到容器中的空/auth下，使容器使用宿主机上生成的文件
      REGISTRY_AUTH 、REGISTRY_AUTH_HTPASSWD_REALM 验证方式，固定写法
      REGISTRY_AUTH_HTPASSWD_PATH：容器中验证文件的路径，要写文件的路径
    # 第四步：重命名需要上传的镜像，在本地服务器上执行
      docker tag IMAGES ip:port/IMAGES_NAME   # 必须重命名为这种格式
    # 第五步：利用docker push 上传刚才重命名的镜像
      docker push ip:port/IMAGES_NAME

### 1. 查看私有仓库中的镜像
 ######1.1 在本地shell中可以通过curl进行查询，也可以在浏览器中通过http://ip:port/v2/_catalog进行访问：
    无认证的私有仓库：
        curl -X GET http://ip:port/v2/_catalog
    带认证的私有仓库
      curl -u username:password -X GET http://ip:port/v2/_catalog
      带认证的不带-u参数，因验证不通过会报如下错误：
        {"errors":[{"code":"UNAUTHORIZED","message":"authentication required","detail":[{"Type":"registry","Class":"","Name":"catalog","Action":"*"}]}]}

###### 1.2 获取获取某个镜像的标签列表,也可以在浏览其中通过http:/v2/image_name/tags/list进行访问：
    无认证的私有仓库：
        curl -X GET http://ip:port/v2/image_name/tags/list
    带认证的私有仓库
      curl -u username:password -X GET http:/v2/image_name/tags/list

### 2.删除私有仓库中的镜像
    第一步：查看digest
        curl -I --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -u dongxiaojian:dongxiaojian -X GET  http://ip:port/v2/centos_test/manifests/latest
        在返回值中的Docker-Content-Digest的值就是digest
    第二步：删除镜像
        curl -u dongxiaojian:dongxiaojian -X DELETE ip:port/v2/centos_test/manifests/sha:365fc7f33107869dfcf2b3ba220ce0aa42e16d3f8e8b3c21d72af1ee622f0cf0
        这一步不一定成功，可能会抛出 {"errors":[{"code":"UNSUPPORTED","message":"The operation is unsupported."}]}异常
        解决方案：在启动registry时指定以下环境变量：-e REGISTRY_STORAGE_DELETE_ENABLED=true
    第三步：进行垃圾回收
      看似已经删除了，其实硬盘地址并没有释放。是因为docker删除image只是删除的image的元数据信息。层数据并没有删除。现在进入registry中进行垃圾回收
      这里先坑着，稍后补充。可以参考博客：http://blog.51cto.com/302876016/1966816


###### *注：其中带-u 都是带认证私有仓库的操作，如果是不需要认证的仓库，把这个参数去掉就行。关于删除镜像，进行垃圾回收大都是通过yum文件进行操作的，待我整体学习完了之后再回来填坑*
