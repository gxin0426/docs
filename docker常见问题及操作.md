

### 0.docker原理

####1.docker是一个c/s架构主要组件包括

1. 常驻后台进程dockerd
2. 一个和dockerd交互的rest api server
3. 命令行cli接口 通过和rest api进行交互

![](dockerimage\dockeraxiom.png)

- docker daemon : dockerd用来监听docker api的请求和管理docker镜像 容器 网络 卷
- docker client 是交互工具 如docker run  docker build等命令
- image： 镜像是一个制度模板 带有镜像的说明 
- container： 容器是一个镜像的可运行的实例 可以使用rest api或者cli来操作容器 容器的实质是进程 但与直接在宿主执行的实例进程不同，容器进程属于自己独立的namespace 因此容器用于自己的root文件系统 网络配置 进程空间 用户id 

文章链接：  https://i4t.com/4248.html 

### 1.docker常见问题

- ![](dockerimage\问题1.png)

- 原因：此linux内核中的selinux不支持overlay2 graph driver ，解决办法有两个

  - 启动一个新内核

  - 在docker里禁用selinux（--selinux-enabled=false）

    修改 /etc/sysconfig/docker中的--selinux-enabled=false

  - 重启docker

### 2.docker 自带仓库 registry

- 启动docker仓库容器

  ```she
  docker run -d -p 5000:5000 --restart=always --name registry registry
  ```

- 指定本地路径

  ```she
  $ docker run -d \
      -p 5000:5000 \
      -v /opt/data/registry:/var/lib/registry \
      registry
  ```

- 打标记

  ```she
  $ docker tag centos:latest 127.0.0.1:5000/centos:latest
  ```

- 使用 `docker push` 上传标记的镜像 

  ```she
  $ docker push 127.0.0.1:5000/centos:latest
  ```

  

- docker登录Harbor

  ```she
  docker login 10.2.21.11:32023  #输入用户名密码
  
  #1.标记镜像
  docker tag {镜像名}:{tag} {Harbor地址}:{端口}/{Harbor项目名}/{自定义镜像名}:{自定义tag}
  #eg:docker tag vmware/harbor-adminserver:v1.1.0 192.168.2.108:5000/test/harbor-adminserver:v1.1.0
  
  #2.push 到Harbor
  docker push {Harbor地址}:{端口}/{自定义镜像名}:{自定义tag}
  #eg:docker push 192.168.2.108:5000/test/harbor-adminserver:v1.1.0
  
  3.pull 到本地
  docker pull 192.168.2.108：5000/test/harbor-adminserver:v1.1.0
  
  ```

  

  

### 3.docker 命令

#### 1.docker基本命令

1. **attach**
   1. 
2. **build**
   1. --rm=false 不删除临时镜像
   2. -f 使用自定义Dockerfile
   3. -t 添加标签
3. **commit**
   1. -a --autohr作者
   2. -c --change 改变参数  可更改参数：CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR
   3. -m --message 提交一个commit备注信息
   4. -pause=false or true 在执行commit操作时 容器内所有进程是处于暂停状态 false表示不暂停
4. **cp**
   1. docker cp <containerid>:/tmp/a.txt $(home)/dockerdata
5. **create**
   1. create命令只是创建一个container 并且在容器文件层的最上面添加一个读写层 容器里所有数据的变化都发生在读写层 当使用create命令后 在使用start命令启动这个容器
6. **diff**
7. **events**
   1. docker events --since '2020-07-01' or '2020-07-01T15:49:30'
   2. docker events --filter 'event=stop'
   3. docker events --filter 'container=id'
8. **exec**
   1. exec是容器内部执行命令的命令 在docker使用过程中 很少出现启动容器时直接创建pty的情况 因为在启动容器时创建pty 退出终端时 容器也会关闭
   2. docker exec id ps -ef 
   3. docker exec id touch /tmp/a.sh
9. **logs**
   1. docker logs -f log不会退出 容器新产生的日志会显示出来
10. **rename**
    1. 重命名
11. **run**
    1. **docker run --pid=host reh17 strace -p 1234 xxxxxx** 通过这个命令可以让新容器访问主机pid=1234的容器的strace进程了（其他还包括 --ipc --uts）
    2. **docker run -d --cidfile=/tmp/id.log busybox** 通过指定cidfile docker运行后 将id写入cidfile中
    3. --**restart** : no always on-failure(在容器遭遇异常原因退出时--restart=on-failure:50)查看重启次数 **docker inspect -f '{{.HostConfig.RestartPolicy}}' mybusybox**
    4. **a**.--dns 设置一个dns服务器为容器（默认情况复用主机dns）**b**.--net **c**.--add-host **d**.--mac-address
    5. **内存 cpu等限制**
       1. -m 300M --memory-swap -1 : --memory-swap没有限制 默认 memory+swap=2*memory
       2. -m 300M --memory-swap 1G :  允许swap+memory 等于1G
       3. 如果容器进程出现oom docker会强制杀死错误进程 想关闭此功能 使用 --oom-kill-disable 使用ci功能时 容器要内存限制 否则有可能耗尽主机内存 非常危险
    6. docker run --privileged 那么docker将允许访问主机除了AppArmor和selinux之外的所有进程
12. **start**
    1. docker start -a xxxxx --attach docker会把容器中的stderr和stdout重定向到主机的stderr和stdout流中
13. **stats**
    1. 监控查看容器资源的命令 stats命令可以统计cpu使用率 内存使用率 网络吞吐量
    2. docker stats --no-stream a68fc01cbe4e 
14. 

### 4.容器运行时 安全工具

- 开源工具

![](dockerimage\docker安全工具.png)

- 主流供应商

![](dockerimage\docker安全主流供应商.png)]

### 5.docker 的空间使用分析和清理

~~~shell
#根据使用的存储驱动的不同，相应目录会有所不同
$du -h --max-depth=1 |sort
#Docker 的内置 CLI 指令 docker system df ，可用于查询镜像（Images）、容器（Containers）和本地卷（Local Volumes）等空间使用大户的空间占用情况
$docker system df
#可以进一步通过 -v 参数查看空间占用细节，以确定具体是哪个镜像、容器或本地卷占用了过高空间。示例输出如下
$docker system df -v
#清理空间 
$docker system prune
WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all images without at least one container associated to them
        - all build cache
####手动清理

#镜像清理
# 删除所有悬空镜像，但不会删除未使用镜像
$docker rmi $(docker images -f "dangling=true" -q)
# 删除所有未使用镜像和悬空镜像。
# 【说明】：轮询到还在被使用的镜像时，会有类似"image is being used by xxx container"的告警信息，所以相关镜像不会被删除，忽略即可
$docker rmi $(docker images -q)

#卷清理
$docker system df -v #查看卷 对于未被任何容器调用的卷（-v 结果信息中，"LINKS" 显示为 0）
# 删除所有未被任何容器关联引用的卷：
$docker volume rm $(docker volume ls -qf dangling=true)

# 也可以直接使用如下指令，删除所有未被任何容器关联引用的卷（但建议使用上面的方式）
# 【说明】轮询到还在使用的卷时，会有类似"volume is in use"的告警信息，所以相关卷不会被删除，忽略即可。
$docker volume rm $(docker volume ls -q)

#容器清理
# 删除所有已退出的容器
docker rm -v $(docker ps -aq -f status=exited)
# 删除所有状态为 dead 的容器
docker rm -v $(docker ps -aq -f status=dead)

$docker ps -s #分析容器占用空间 
# 如下容器的原始镜像占用了422MB空间，实际运行过程中只占用了2B空间：
CONTAINERID  IMAGE   COMMAND  CREATED   STATUS  PORTS   NAMES        SIZE
ac3912      word    ntrypoin      3      11   80/tcp  Web_web_4  2B(virtual422MB)

#使用Device Mapper 存储驱动限制容器磁盘空间 如果使用device mapper 作为底层驱动 则可以通过Docker daemon 参数 全局限制单个容器的空间大小
--storage-opt dm.basesize=20G 

#使用btrfs   btrfs驱动主要使用btrfs所提供的的subvolume功能实现
$ btrfs qgroup limit -e 50G /var/lib/docker/btrfs/subvolumes/<CONTAINER_ID>

#文章链接：https://yq.aliyun.com/articles/272173（其中包括 如何给容器服务的Docker增加数据盘）
~~~

### 6.载出和载入镜像

~~~shell
$ docker save -o calico_node_v3.8.2.tar calico/node:v3.8.2
$ docker load --input calico_node_v3.8.2.tar
~~~

### 7.docker-compose network网段设置

~~~yaml
# docker version:  18.06.0+
# docker-compose version: 1.23.2+
# OpenSSL version: OpenSSL 1.1.0h
version: "3.7"
services:
  web:
    image: alenx/walle-web:2.1
    container_name: walle-nginx
    hostname: nginx-web
    ports:
      # 如果宿主机80端口被占用，可自行修改为其他port(>=1024)
      # 0.0.0.0:要绑定的宿主机端口:docker容器内端口80
      - "80:80"
    depends_on:
      - python
    networks:
      - walle-net
    restart: always

  python:
    image: alenx/walle-python:2.1
    container_name: walle-python
    hostname: walle-python
    env_file:
      # walle.env需和docker-compose在同级目录
      - ./walle.env
    command: bash -c "cd /opt/walle_home/ && /bin/bash admin.sh migration &&  python waller.py"
    expose:
      - "5000"
    volumes:
      - /opt/walle_home/plugins/:/opt/walle_home/plugins/
      - /opt/walle_home/codebase/:/opt/walle_home/codebase/
      - /opt/walle_home/logs/:/opt/walle_home/logs/
      - /root/.ssh:/root/.ssh/
    depends_on:
      - db
    networks:
      - walle-net
    restart: always

  db:
    image: mysql
    container_name: walle-mysql
    hostname: walle-mysql
    env_file:
      - ./walle.env
    command: [ '--default-authentication-plugin=mysql_native_password', '--character-set-server=utf8mb4', '--collation-server=utf8mb4_unicode_ci']
    ports:
      - "3306:3306"
    expose:
      - "3306"
    volumes:
      - /data/walle/mysql:/var/lib/mysql
    networks:
      - walle-net
    restart: always

networks:
  walle-net
    driver: bridge 
    ipam: 
      config:  
        - subnet: 172.33.255.1/24
~~~

### 8.docker daemon 远程访问

#### 1.docker daemon的连接方式

1. Unix域套接字
   1. 默认方式 会生成一个 /var/run/docker.sock文件 Unix域套接字用于本地进程间的通信
2. tcp端口监听
   1. 服务端开启端口监听 dockerd -H IP:port 客户端通过docker -H IP:port 访问 （不安全）

#### 2.更改docker配置

- 在/etc/docker/daemon.json 中添加如下内容（现在docker的配置信息都在daemon文件中配置 没有需要创建）

~~~shell
{
  "hosts" : ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"]
}
#"unix:///var/run/docker.sock" unix socket ：本地客户端通过这个连接docker daemon
#"tcp://0.0.0.0:2375" tcp  ： 表示允许任何远程客户端通过2375访问（2376加密）
~~~

#### 3.更改系统服务配置

- 通过**systemctl edit docker** 来调用文本编辑器修改单元 新建或者修改  /etc/systemd/system/docker.service.d/override.conf 

~~~shell
##Add this to the file for the docker daemon to use different ExecStart parameters (more things can be added here)
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
~~~

- 默认情况下使用systemd时， docker.service的设置是 ExecStart=/usr/bin/dockerd -H fd:// ，这将覆盖daemon.json中的任何hosts 通过override.conf 文件将ExecStart=/usr/bin/dockerd 这将会使用在daemon.json中设置的hosts 这个文件中的第一行必须有 ExecStart= 用于清除默认的ExecStart参数

#### 4.重新加载daemon和重启docker服务

~~~shell
systemctl daemon-reload
systemctl restart docker
~~~

