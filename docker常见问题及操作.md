

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

  

  

### 3.常见报错

```go
import fmt 改成 import "fmt"
报错：
package main:
1.go:2:11: expected 'STRING', found newline
1.go:4:1: expected ';', found 'func'
```

unexpected semicolon or newline before 

```go
package main
import "fmt"

func main()
{
    fmt.Println("hello.")
}
```

报错：

```go
# command-line-arguments
.\1.go:5: syntax error: unexpected semicolon or newline before {
```

- 区分大小写

代码：

```
package main
import "fmt"

func main(){
    fmt.println("hello.")
}
```

报错：

```
# command-line-arguments  
.\1.go:5: cannot refer to unexported name fmt.println  
.\1.go:5: undefined: fmt.println  
```

正确写法： 

```
package main
import "fmt"

func main(){
    fmt.Println("hello.")
}
```

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
$docker system df -v #查看容器 对于未被任何容器调用的卷（-v 结果信息中，"LINKS" 显示为 0）
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

