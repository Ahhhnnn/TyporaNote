# Nginx入门

Nginx是一款轻量级的Web服务器、反向代理服务器，由于它的内存占用少，启动极快，高并发能力强，在互联网项目中广泛应用。

![image-20200404192611705](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200404192611705.png)

## 正向代理

由于防火墙的原因，我们并不能直接访问谷歌，那么我们可以借助VPN来实现，这就是一个简单的正向代理的例子。这里你能够发现，正向代理“代理”的是客户端，而且客户端是知道目标的，而目标是不知道客户端是通过VPN访问的。

可以理解为代理在client端，为正向代理。如果在server端，则为反向代理。

## 反向代理

当我们在外网访问百度的时候，其实会进行一个转发，代理到内网去，这就是所谓的反向代理，即反向代理“代理”的是服务器端，而且这一个过程对于客户端而言是透明的。



## 负载均衡

反向代理服务器将大量请求分发到不同的web服务器上。



## 动静分离

通常静态资源和动态资源都部署在反向代理服务器之后的web服务器上，为了提高响应速度，可以将资源分离，将静态资源放在nginx进行管理，便于快速返回。



## Centos 安装nginx

使用yum命令安装nginx

```
sudo yum install nginx
```

设置开机启动

```
sudo systemctl enable nginx
```

启动nginx

```
sudo systemctl start nginx
```

期间启动失败，由于nginx.conf 默认使用80端口，而我的80端口被占用了，所以需要手动修改配置，然后重启。

检查nginx运行状态

```
sudo systemctl status nginx
```

修改nginx.conf 监听xx.xxx.xx.xx 的8083端口

将请求转发到本机的tomcat应用上地址为 127.0.0.1:8000/weacherSec



启动服务常用命令

```
启动一个服务：systemctl start firewalld.service
关闭一个服务：systemctl stop firewalld.service
重启一个服务：systemctl restart firewalld.service
显示一个服务的状态：systemctl status firewalld.service
在开机时启用一个服务：systemctl enable firewalld.service
在开机时禁用一个服务：systemctl disable firewalld.service
```



## 常用命令

必须进入到nginx的sbin目录下执行

![image-20200404194132945](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200404194132945.png)

![image-20200404194200556](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200404194200556.png)

重新加载nginx配置文件,不重新启动nginx

./nginx -s reload



## Nginx 配置文件

### 全局块

配置文件开始到events之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令，主要有nginx服务器的用户组，允许生成的worker process数，进程pid存放路径，日志存放路径等

```
#user  nobody;
worker_processes  1; #关键配置，表示处理并发的数量，一般和cpu数有关
```



## Event块

主要影响nginx服务器与用户的网络连接，常用设置包括是否开启对多work process下的网络连接进行序列化，是否允许同时接受多个网络连接，选取那种事件驱动模型处理连接其你去，每个work process可以同时支持的最大连接数等信息。

```
events {
    worker_connections  1024;
}
```

表示每个work process进程，支持最大1024个连接，这部分配置对nginx性能影响很大，需要灵活配置。

## http 块

nginx中服务器配置最频繁的部分，代理、缓存、日志定义等大部分功能和第三方模块配置都在这里

注意：http块也可以包括http全局快、server块。

### http全局块

包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单链接请求数上限等

```
include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
```



### server 块



```
server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
```



## 反向代理实例1

修改nginx.conf配置文件,修改nginx监听端口、服务地址、转发地址等

![image-20200405152124559](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405152124559.png)

将 /weather 转发到第一个weather项目

备注：因为我的vue项目打包时使用的是history 模式部署 （ vue的路由模式如果使用history，刷新会报404错误。）

将/weatherSec 转发到第二个weather项目

完全匹配模式：

![image-20200405153911094](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405153911094.png)



localtion**匹配模式**

![image-20200405154254658](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405154254658.png)





## 负载均衡实例

在nginx.conf进行配置

增加一个 upstream 配置

然后再server块 location配置中使用上面创建的upstream配置

![image-20200405155411295](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405155411295.png)

![image-20200405155602298](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405155602298.png)

nginx 负载均衡 分配策略

### 轮询 （默认）

![image-20200405155952196](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405155952196.png)

### weight 权重，设置权重

默认权重为1，若权重越高，所分配的次数越多，概率越大。

在upstream中配置server时，在ip地址后进行配置

###  ip hash

每个请求根据访问的ip进行hash结果分配，保证每次访问的后端服务器都是同一个，可以解决session等问题。

![image-20200405160340266](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405160340266.png)

### fair 第三方

根据后端服务器的响应时间进行分配，响应时间短的优先分配

![image-20200405160419659](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405160419659.png)



## 动静分离实例

动静分离结构图

![image-20200405164947500](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405164947500.png)

在ngnix.conf 配置文件中进行配置

同样是在location节点进行配置 转发

![image-20200405164533283](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405164533283.png)

/www/ 和 /image/ ：请求的路径

root /data/; ：静态资源文件夹

autoindex on ：当从浏览器访问文件夹时，自动列出文件夹下的内容

expires 3d: 过期时间，ngnix会设置静态资源在浏览器的过期时间

当路径中包含www 和 image时，会请求道data静态资源文件夹中



## 配置Nginx的高可用集群

通过keepalived 实现高可用。

所有的请求不会直接到nginx，而是全部打到一个虚拟ip上，由keepalived去检测nginx的可用状态，再讲请求分发到nginx上，以实现高可用的效果。

![image-20200405165715959](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405165715959.png)



### 安装keepalived

```
yum install keepalived -y
```

![image-20200405170313622](C:\Users\YMXD\AppData\Roaming\Typora\typora-user-images\image-20200405170313622.png)

生成了一个keepalived.conf 配置文件



### 完成高可用配置

需要分别配置master 和 backup的keepalived配置文件

注意：网卡可通过 ifconfig命令查看  

​			通过if a 命令查看网卡绑定的虚拟ip

master

```
global_defs {
    notification_email {
        997914490@qq.com
    }
    notification_email_from sns-lvs@gmail.com
    smtp_server smtp.hysec.com
    smtp_connection_timeout 30
    router_id nginx_master        # 设置nginx master的id，在一个网络应该是唯一的
}
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh"    #最后手动执行下此脚本，以确保此脚本能够正常执行
    interval 2                          #（检测脚本执行的间隔，单位是秒）
    weight 2
}
vrrp_instance VI_1 {
    state MASTER            # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth0            # 当前进行vrrp通讯的网络接口卡(当前centos的网卡)
    virtual_router_id 66        # 虚拟路由编号，主从要一直
    priority 100            # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1            # 检查间隔，默认为1s(vrrp组播周期秒数)
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
    chk_http_port            #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.0.200            # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

backup

```
global_defs {
    notification_email {
        997914490@qq.com
    }
    notification_email_from sns-lvs@gmail.com
    smtp_server smtp.hysec.com
    smtp_connection_timeout 30
    router_id nginx_backup              # 设置nginx backup的id，在一个网络应该是唯一的
}
vrrp_script chk_http_port {
    script "/usr/local/src/check_nginx_pid.sh"
    interval 2                          #（检测脚本执行的间隔）
    weight 2
}
vrrp_instance VI_1 {
    state BACKUP                        # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth0                      # 当前进行vrrp通讯的网络接口卡(当前centos的网卡)
    virtual_router_id 66                # 虚拟路由编号，主从要一直
    priority 99                         # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1                        # 检查间隔，默认为1s(vrrp组播周期秒数)
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_http_port                   #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.0.200                   # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

编写/usr/local/src/check_nginx_pid.sh 下的检测脚本

check_nginx_pid.sh

```
#!/bin/bash
A=`ps -C nginx --no-header |wc -l`        
if [ $A -eq 0 ];then                            
    /usr/local/nginx/sbin/nginx                #重启nginx
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then    #nginx重启失败
        exit 1
    else
        exit 0
    fi
else
    exit 0
fi
```

