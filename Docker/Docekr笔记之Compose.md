`Compose`是一个用于定义和运行多个容器的Docker应用程序的工具。使用Compose，需要使用YAML文件来配置应用程序的服务，所以主要的是怎么编写配置文件才是关键的。
### 0. 安装Compose  [查看更多](https://docs.docker.com/compose/install/#install-compose)
###### 在windows、mac上不需要安装，需要在Linux上安装
    # 其实就两条命令，可以在https://github.com/docker/compose/releases 查看对应的版本号
    curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
### 1.Docker Compose File版本
Docker Compose File 有多个版本，基本都是向后兼容的，在YAML文件中，开头用version关键词表明当前Docker Compose File使用的版本。
![图片.png](https://upload-images.jianshu.io/upload_images/11227136-3988a85615e622be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###### Docker Compose File的命令略多，学习起来也是不当便，可以参考 [官方文档](https://docs.docker.com/compose/compose-file/#service-configuration-reference)。下边一个小栗子。

### 2.Docker Compose小栗子
准备用flask搭建web服务来练习Docker Compose。
###### 第一步：准备
    1. 创建项目目录
        mkdir web_docker
        cd web_docker
        mkdir web_code
    2.在子目录web_code下创建flask代码文件test_app.py，内容就是简单的hello world
        import flask
        app = flask.Flask(__name__)
        @app.route('/')
        def main():
            return "hello word"
        if __name__ == "__main__":
            app.run(host="0.0.0.0", port=5000, debug=True)
      
      # 这里需要注意的是,app.run的参数host一定要指定为“0.0.0.0”否则无法通过外部访问flask

######  第二步：编写Dockerfile以及docker-compose.yaml
        1. 在项目目录web_docker下编写Dockerfile
        # web docekr images
        FROM python:alpine3.6
        COPY ./web_code /code
        RUN pip install flask
        WORKDIR /code
        CMD ["python","test_app.py"]
    注： 关于python版本直接在python后边加是没有用的，需要通过tag信息才可以，具体的版本对应的信息可以查看：https://github.com/docker-library/docs/tree/master/python

    2. 在项目目录web_docker下编写docker-compose.yaml
      version: "3.6"
      services:
        flask_web:
          build:
            context: /home/dong/web_docker
          ports:
            - "5000:5000"
          networks:
            - web
      networks:
        web:
          driver: "bridge"


      
###### 第三步：通过docker-compose启动：
       1. 通过docker-compose config检查编写YAML文件是否语法正确
       2.如果正确，则执行  docker-compose  up  
###### 第四步： 检查
      在宿主机上，执行 curl 127.0.0.1:5000  如果返回 hello world 表明已经成功了


###### 第五步：打完收工

###### *这个小栗子主要是体会下docker compose使用的流程，但是在生产环境下还是远远不够的，还需要多练，尤其时命令以及编写格式。*





























