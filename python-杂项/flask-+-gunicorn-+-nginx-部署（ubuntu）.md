# flask + gunicorn + nginx 部署（ubuntu）

######  建议先全部看完， 形成一个整体的流程概念再进行安装。本人水平有限，如有错误欢迎指正。




###    1.安装依赖

    安装gcc， g++依赖：
       apt-get install build-essential
       apt-get install libtool 
    
	安装 pcre依赖库
      sudo apt-get update
      sudo apt-get install libpcre3 libpcre3-dev

    安装 zlib依赖库
      apt-get install zlib1g-dev

    安装 ssl依赖库
    apt-get install openssl

###   2.安装nginx（[http://nginx.org](http://nginx.org)） 

	下载源码（因版本号不同而有差异，以下同）：
	 	wget http://nginx.org/download/nginx-1.11.3.tar.gz

	解压：
		tar -zxvf nginx-1.11.3.tar.gz

	进入解压目录：
		cd nginx-1.11.3

	配置：
		./configure --prefix=/usr/local/nginx
 
	编辑nginx(此处可能有错误，如报错请自查)：
		make
		
	安装nginx：
		sudo make install

	启动nginx：
		
	注意：-c 指定配置文件的路径，不加的话，nginx会自动加载默认路径的配置文件，可以通过 -h查看帮助命令。
	
	查看nginx进程：
		ps -ajx|grep nginx

	
进行到此处，可能用到的命令`whereis nginx`用来查找文件nginx的文件路径

#####nginx的命令（网上会有很多，只说一下我执行ok的命令）：
    启动： sudo /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
		
	停止： sudo /usr/local/nginx/sbin/nginx -s -stop

	重载： sudo /usr/local/nginx/sbin/nginx -s reload

	检查配置文件是否正确： nginx -t

### 3. 配置nginx配置文件
 配置文件在 `/usr/local/nginx/conf`文件夹下`nginx.conf`


		server {                                                                       
				listen 80;                   # 服务器监听端口                                                 
				server_name 110.110.110.110; # 这里写你的域名或者公网IP                                                    
				charset      utf-8;          # 编码                                                  
				client_max_body_size 75M;    # 之前写的关于GET和POST的区别，这里应该是原因之一吧， 显示最大的请求体                                                                                                           
    	   location / {
	            proxy_pass http://127.0.0.1:5002;#转发到gunicorn的IP和端口号
	            proxy_set_header Host $host; #下边这两个我也不知道是干嘛的
	            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;            
			}                                                                              
			}



	因为我只是为接口提供服务，使用的是http协议，这个配置是在http协议下的，结构为http>server>location.如果使用https协议请参考网络。
	

	检查配置文件是否正确： nginx -t


### 4.安装supervisor

   `sudo apt-get install supervisor` 也可以通过`pip install supervisor` 进行安装（一般在虚拟环境中这样进行安装）
#######  *可能需要的依赖：meld3  elementtree（我在ubuntu中没遇到，centos7中遇到了）

### 5. supervisor的配置文件
	`echo_supervisord_conf > supervisor.conf `生成supervisor的默认配置文件，

    [program:myapp]
	command= gunicorn命令的路径 -w4 -b0.0.0.0:8000 myapp:app    ; supervisor启动命令
	directory=项目路径                                                 ; 项目的文件夹路径
	startsecs=0                                                                             ; 启动时间
	stopwaitsecs=0                                                                          ; 终止等待时间
	autostart=false                                                                         ; 是否自动启动
	utorestart=false                                                                        ; 是否自动重启
	stdout_logfile=log日志路径                           ; log 日志
	stderr_logfile=错误日志路径                           ; 错误日志

#####  其中的 directory 可以省去， 直接写到command中去，在配置ubuntu时没有遇到问题，在配置centos时有报错。`command= gunicorn的路径   flask的app.py的路径 -w4 -b0.0.0.0:8000 myapp:app`

	supervisor的常用命令：

	supervisord -c supervisor.conf                             通过配置文件启动supervisor
	supervisorctl -c supervisor.conf status                    察看supervisor的状态
	supervisorctl -c supervisor.conf reload                    重新载入 配置文件
	supervisorctl -c supervisor.conf start [all]|[appname]     启动指定/所有 supervisor管理的程序进程
	supervisorctl -c supervisor.conf stop [all]|[appname]      关闭指定/所有 supervisor管理的程序进程

### 6. 安装gunicorn

   	安装命令：pip install gunicorn
	启动命令：gunicorn -w4 -b127.0.0.1:5002 myapp:app



##### 总结：其实我认为比较好的安装顺序应该先从gunicorn安装起，因为它最简单，安装完后可以通过命令直接测试，不需要写配置文件。安装完gunicorn以后安装supervisor，写好配置文件，启动测试。最后安装nginx。这些东西写起来简单，真正做起来有点麻烦，要根据相应的实际生产环境做相应的调整。

*注： 本人水平有限， 如有错误欢迎提出指正！如有引用， 请注明出处！！*

