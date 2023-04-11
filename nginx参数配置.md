## nginx参数配置

```yaml
#部署
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 1
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: jd-train-cn-north-1-inner.jcr.service.jdcloud.com/9n-train:nginx_latest
        volumeMounts:
        - mountPath: /etc/nginx/nginx.conf
          name: nginxconf
          subPath: nginx.conf
        resources:
          limits:
            cpu: "1"
            memory: 512Mi
          requests:
            cpu: "1"
            memory: 512Mi
      volumes:
      - name: nginxconf
        configMap:
          name: nginxconf
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  selector:
    run: my-nginx
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-nginx
  namespace: module-hub
spec:
  rules:
  - host: 9n-das.jd.com
    http:
      paths:
      - backend:
          serviceName: my-nginx
          servicePort: 80
        path: /
```

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /run/nginx.pid;

events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        location /feature-center {
                proxy_pass http://feature-center-console-uat.jd.com/;
#                proxy_redirect off;
#                proxy_set_header Host $host;
#                proxy_set_header X-Real-IP $remote_addr;
#                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                add_header Access-Control-Allow-Origin "http://xxx.jd.com:8080" always;
                add_header Access-Control-Allow-Methods "POST, GET, DELETE, PUT, OPTIONS" always;
                add_header Access-Control-Allow-Headers "Content-type,Content-Length,Authorization,Origin,Access-Control-Allow-Origin,Accept,Options,X-Requested-With" always;
                add_header Access-Control-Allow-Credentials true always;
                add_header Access-Control-Max-Age: 3600;

                if ($request_method = 'OPTIONS') {
                        return 204;
                }
        }
    }
}
```

```shell
# configmap创建方式
# 创建单个文件的cm
kubectl create cm my-config --from-file=path/file

# 创建一个文件夹下所有文件 cm
kubectl create cm my-config --from-file=[key1=]/path/file1 --from-file=[key2=]/path/file2

# 键值对创建cm
kubectl create cm my-config --from-literal=mykey1=nihao --from-literal=mykey2=nihao2

#通过键值对文件创建cm
kubectl create cm my-config  --from-env-file=path/file
cat path/file
a=1
b=2
c=3
```

```shell
kill -HUP pid #让进程挂起 重新加载配置
kill -9       #六亲不认的杀掉
term.         #正常退出的进程
```

### nginx配置结构

#### 1.全局块

`worker_processes 1;`

worker_processes 值越大 可以支持的并发处理越多



```shell
# 指定可以运行nginx服务的用户和用户组，只能在全局块配置
# user [user] [group]
# 将user指令注释掉，或者配置成nobody的话所有用户都可以运行
# user nobody nobody;
# user指令在Windows上不生效，如果你制定具体用户和用户组会报小面警告
# nginx: [warn] "user" is not supported, ignored in D:\software\nginx-1.18.0/conf/nginx.conf:2

# 指定工作线程数，可以制定具体的进程数，也可使用自动模式，这个指令只能在全局块配置
# worker_processes number | auto；
# 列子：指定4个工作线程，这种情况下会生成一个master进程和4个worker进程
# worker_processes 4;

# 指定pid文件存放的路径，这个指令只能在全局块配置
# pid logs/nginx.pid;

# 指定错误日志的路径和日志级别，此指令可以在全局块、http块、server块以及location块中配置。(在不同的块配置有啥区别？？)
# 其中debug级别的日志需要编译时使用--with-debug开启debug开关
# error_log [path] [debug | info | notice | warn | error | crit | alert | emerg] 
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;
```

#### 2.events块

```ini
events{
	work_connections 1024;
}
```

events块涉及的指令主要影响nginx服务器与用户的网络连接，常用的设置包括开启对多work process下的网络连接进行序列化 是否允许同时接收多个网络连接， 每个work process可以同时支持的最大连接数

```nginx
# 当某一时刻只有一个网络连接到来时，多个睡眠进程会被同时叫醒，但只有一个进程可获得连接。如果每次唤醒的进程数目太多，会影响一部分系统性能。在Nginx服务器的多进程下，就有可能出现这样的问题。
# 开启的时候，将会对多个Nginx进程接收连接进行序列化，防止多个进程对连接的争抢
# 默认是开启状态，只能在events块中进行配置
# accept_mutex on | off;

# 如果multi_accept被禁止了，nginx一个工作进程只能同时接受一个新的连接。否则，一个工作进程可以同时接受所有的新连接。 
# 如果nginx使用kqueue连接方法，那么这条指令会被忽略，因为这个方法会报告在等待被接受的新连接的数量。
# 默认是off状态，只能在event块配置
# multi_accept on | off;

# 指定使用哪种网络IO模型，method可选择的内容有：select、poll、kqueue、epoll、rtsig、/dev/poll以及eventport，一般操作系统不是支持上面所有模型的。
# 只能在events块中进行配置
# use method
# use epoll

# 设置允许每一个worker process同时开启的最大连接数，当每个工作进程接受的连接数超过这个值时将不再接收连接
# 当所有的工作进程都接收满时，连接进入logback，logback满后连接被拒绝
# 只能在events块中进行配置
# 注意：这个值不能超过超过系统支持打开的最大文件数，也不能超过单个进程支持打开的最大文件数，具体可以参考这篇文章：https://cloud.tencent.com/developer/article/1114773
# worker_connections  1024;
```

#### 3.http块

```nginx
 http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

- http 块

http全局块配置包括文件引入、mime-type定义、日志自定义、连接超时时间、单连接请求数上限

```nginx
#参考文章
# https://www.cnblogs.com/54chensongxia/p/12938929.html
# 常用的浏览器中，可以显示的内容有HTML、XML、GIF及Flash等种类繁多的文本、媒体等资源，浏览器为区分这些资源，需要使用MIME Type。换言之，MIME Type是网络资源的媒体类型。Nginx服务器作为Web服务器，必须能够识别前端请求的资源类型。

# include指令，用于包含其他的配置文件，可以放在配置文件的任何地方，但是要注意你包含进来的配置文件一定符合配置规范，比如说你include进来的配置是worker_processes指令的配置，而你将这个指令包含到了http块中，着肯定是不行的，上面已经介绍过worker_processes指令只能在全局块中。
# 下面的指令将mime.types包含进来，mime.types和ngin.cfg同级目录，不同级的话需要指定具体路径
# include  mime.types;

# 配置默认类型，如果不加此指令，默认值为text/plain。
# 此指令还可以在http块、server块或者location块中进行配置。
# default_type  application/octet-stream;

# access_log配置，此指令可以在http块、server块或者location块中进行设置
# 在全局块中，我们介绍过errer_log指令，其用于配置Nginx进程运行时的日志存放和级别，此处所指的日志与常规的不同，它是指记录Nginx服务器提供服务过程应答前端请求的日志
# access_log path [format [buffer=size]]
# 如果你要关闭access_log,你可以使用下面的命令
# access_log off;

# log_format指令，用于定义日志格式，此指令只能在http块中进行配置
# log_format  main '$remote_addr - $remote_user [$time_local] "$request" '
#                  '$status $body_bytes_sent "$http_referer" '
#                  '"$http_user_agent" "$http_x_forwarded_for"';
# 定义了上面的日志格式后，可以以下面的形式使用日志
# access_log  logs/access.log  main;

# 开启关闭sendfile方式传输文件，可以在http块、server块或者location块中进行配置
# sendfile  on | off;

# 设置sendfile最大数据量,此指令可以在http块、server块或location块中配置
# sendfile_max_chunk size;
# 其中，size值如果大于0，Nginx进程的每个worker process每次调用sendfile()传输的数据量最大不能超过这个值(这里是128k，所以每次不能超过128k)；如果设置为0，则无限制。默认值为0。
# sendfile_max_chunk 128k;

# 配置连接超时时间,此指令可以在http块、server块或location块中配置。
# 与用户建立会话连接后，Nginx服务器可以保持这些连接打开一段时间
# timeout，服务器端对连接的保持时间。默认值为75s;header_timeout，可选项，在应答报文头部的Keep-Alive域设置超时时间：“Keep-Alive:timeout= header_timeout”。报文中的这个指令可以被Mozilla或者Konqueror识别。
# keepalive_timeout timeout [header_timeout]
# 下面配置的含义是，在服务器端保持连接的时间设置为120 s，发给用户端的应答报文头部中Keep-Alive域的超时时间设置为100 s。
# keepalive_timeout 120s 100s

# 配置单连接请求数上限，此指令可以在http块、server块或location块中配置。
# Nginx服务器端和用户端建立会话连接后，用户端通过此连接发送请求。指令keepalive_requests用于限制用户通过某一连接向Nginx服务器发送请求的次数。默认是100
# keepalive_requests number;
```

- 参考文章：https://www.cnblogs.com/54chensongxia/p/12938929.html

### Location url 加或不加/区别

以访问`http://127.0.0.1:90/proxy/test.html `为例

```nginx
location /proxy/ {
	proxy_pass http://127.0.0.1:81/;
}
#转到 http://127.0.0.1:81/test.html

location  /proxy/ {
	proxy_pass http://127.0.0.1:81;
}
#转到http://127.0.0.1:81/proxy/test.html

location  /proxy/ {
	proxy_pass http://127.0.0.1:81/ftlynx/;
}
#转到http://127.0.0.1:81/ftlynx/test.html

location  /proxy/ {
	proxy_pass http://127.0.0.1:81/ftlynx;
}
#转到 http://127.0.0.1:81/ftlynxtest.html
```

```nginx
localtion / {
    # 所有请求都匹配以下规则
    # 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求
    # xxx 你的配置写在这里
}

location = / {
    # 精确匹配 / ，后面带任何字符串的地址都不符合
}

localtion /api {
    # 匹配任何 /api 开头的URL，包括 /api 后面任意的, 比如 /api/getList
    # 匹配符合以后，还要继续往下搜索
    # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
}

localtion ~ /api/abc {
    # 匹配任何 /api/abc 开头的URL，包括 /api/abc 后面任意的, 比如 /api/abc/getList
    # 匹配符合以后，还要继续往下搜索
    # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
}
```

- 以`/` 通用匹配, 如果没有其它匹配,任何请求都会匹配到
- `=`开头表示精确匹配
- `^~` 开头表示uri以某个常规字符串开头，不是正则匹配
- `~` 开头表示区分大小写的正则匹配
- `~*` 开头表示不区分大小写的正则匹配

#### nginx配置常见问题及配置

#### 问题1（Response to preflight request doesn‘t pass access control check: It does not have HTTP ok status）

调后端接口时会发送两次请求,一次是options ,options通过后再发送get或者post请求,在一开始后端只对请求做了一次拦截,导致前端发送请求时后端只能接收到options请求,并且无token存在,最终解决方法是后端对options做了处理,检测到是option请求时直接放行

#### 问题2 跨域问题

当出现403跨域错误的时候 `No 'Access-Control-Allow-Origin' header is present on the requested resource`，需要给Nginx服务器配置响应的header参数：

```nginx
location / {  
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';

    if ($request_method = 'OPTIONS') {
        return 204;
    }
} 
```

1. Access-Control-Allow-Origin

```shell
默认nginx不允许跨域 给nginx配置 `Access-Control-Allow-Origin *` 后，表示服务器可以接收所有的请求源。即接收所有跨域的访问
```

2. Access-Control-Allow-Headers 防止如下错误：

`Request header field Content-Type is not allowed by Access-Control-Allow-Headers in preflight response.`

这个错误表示当前Content-Type的值不支持。

3. Access-Control-Allow-Methods是为了防止：

`Content-Type is not allowed by Access-Control-Allow-Headers in preflight response.`

4. 给OPTIONS添加204的返回，是为了处理在发送POST请求时Nginx依然拒绝访问

发送‘预检请求’时，需要用到方法OPTIONS，所以服务器需要允许该方法

- 参考文章：https://segmentfault.com/a/1190000012550346





## nginx 尚硅谷笔记

#### 反向代理

#### 负载均衡

#### 动静分离

### 配置文件

#### 1. 全局快

​	从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令，包括运行nginx的用户组、允许生成的 work process数， 进程pid存放路径 日志存放路径和类型以及配置文件的引入等

比如，

```
worker_processes 1;
user nobody;
error_log logs/error.log
pid logs/nginx.pid
```

这是nginx并发处理服务的关键配置，**worker_processes值越大，可以支持的并发处理越多**， 但受硬件限制

#### 2. events 块

events 块涉及的指令主要影响nginx服务器与用户的网络链接，常用的设置包括是否开启对多 work process下的网络连接进行序列化 ， 是否允许同时接收多个网络链接，选取哪种事件驱动模型来处理请求，每个word process可以同时支持的最大连接数

比如下面配置

```
events {
	worker_connections 1024;
}
```

#### 3. http 块

代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里

需要注意： http也可以包括http全局块、 server块

##### http全局块

http全局块配置的指令包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单连接请求数上限

##### server块

这块和虚拟主机关系密切、虚拟主机从用户角度看，和一台独立的硬件主机完全一样，

每个http块可以包括多个server块，而每个server块相当于一个虚拟主机

每个server块也分为全局server块，以及可以同时包括多个location块

###### 全局server块

最常见的配置是本虚拟机的监听配置和本虚拟机的名称和ip配置

#### 反向代理 demo

```shell
#反向代理 demo
http{
	#全局块
	include mime.type;
	default_type application/octet-stream;
	sendfile on;
	keepalive_timeout 65;
	server{
            listen 80;
            server_name localhost;

            location / {
                root html;
                index index.html index.htm;
                proxy_pass http://test:80;
		}
		# 测试
    server{
        listen 9001;
        server_name localhost;

        location ~ /edu/ {

            proxy_pass http://test:8001;
        }
        location ~ /vod/ {
            proxy_pass http://test:8002;
        }
	}
}
```

##### location配置说明

该指令用于匹配url

语法如下：

```
location [= | ~ | ~* | ^~] uri {

}
```

- = : 用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。
- ~ : 用于表示uri包含正则表达式，并区分大小写。
- ~* ： 用于表示uri包含正则表达式，并且不区分大小写。
- ^~ : 用于不含正则表达式的uri前， 要求nginx服务器找到表示uri和请求字符串匹配度最高的location后，立即使用次location处理请求，而不再使用location块中的正则uri和请求字符串匹配。

**注意：如果uri包含正则表达式，则必须要有~ 或者~* 标识**

#### 负载均衡 demo

```
http{
	#全局块
	include mime.type;
	default_type application/octet-stream;
	sendfile on;
	keepalive_timeout 65;
	
	upsteam myserver {
		#fair  weight ip_hash
		ip_hash;
		server test:8080;
		server test:8081;
	}
	
	server{
            listen 80;
            server_name localhost;

            location / {
                root html;
                index index.html index.htm;
                proxy_pass http://myserver;
             }
		}
		# 测试
    server{
        listen 9001;
        server_name localhost;

        location ~ /edu/ {

            proxy_pass http://test:8001;
        }
        location ~ /vod/ {
            proxy_pass http://test:8002;
        }
	}
}
```

#### 动静分离 demo

```
http{
	#全局块
	include mime.type;
	default_type application/octet-stream;
	sendfile on;
	keepalive_timeout 65;
	
	
	server{
            listen 80;
            server_name localhost;
      
            location /www/ {
                root /data;
                index index.html index.htm;
                proxy_pass http://myserver;
             }
             location /image/ {
                root /data/;
                index index.html index.htm;
                proxy_pass http://myserver;
             }
	}
}
```

