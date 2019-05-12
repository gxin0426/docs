

### docker常见问题

- ![](dockerimage\问题1.png)

- 原因：此linux内核中的selinux不支持overlay2 graph driver ，解决办法有两个

  - 启动一个新内核

  - 在docker里禁用selinux（--selinux-enabled=false）

    修改 /etc/sysconfig/docker中的--selinux-enabled=false

  - 重启docker

### docker 自带仓库 registry

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

  

  

### 常见小报错

- ```go
  import fmt 改成 import "fmt"
  报错：
  package main:
  1.go:2:11: expected 'STRING', found newline
  1.go:4:1: expected ';', found 'func'
  ```

- unexpected semicolon or newline before { 

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

### 容器运行时 安全工具

- 开源工具

![](dockerimage\docker安全工具.png)

- 主流供应商

![](dockerimage\docker安全主流供应商.png)]

