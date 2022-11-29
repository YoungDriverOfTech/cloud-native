# 1 SpringCloud
## 1.1 一些组件  
- 服务治理 Eureka
- 服务通信 Ribbon
- 服务通信 Feign
- 服务网关 zuul
- 服务容错 Hystrix
- 服务配置 Config
- 服务监控 Actuator
- 服务跟踪 Zipkin

## 1.2 服务治理  
### 1.2.1 概念  
服务治理的核心有三部分组成：服务提供者，服务消费者，注册中心。  
在分布式系统架构中，每个服务在启动时，将自己的信息存储在注册中心，叫做服务注册。  
服务消费者从注册中心获取服务提供者的网络信息，通过该信息调用服务，叫做服务发现。  
SpringCloud的服务治理使用Eureka来实现，Eureka是Netflix开源的机遇REST的服务治理解决方案，SpringCloud集成了Eureka，提供服务注册和服务发现的功能，可以和基于SpringBoot搭建的微服务应用轻松完成整合，开箱即用，Srping Cloud Eureka。

### 1.2.2 Srping Cloud Eureka  
- Eureka Server 注册中心
- Eureka Client 所有要进行注册的微服务通过Eureka Client连接到Eureka Server，完成注册


# 3. Pod 详解  
## 3.1 概述

### 3.1.1 全部资源清单  
```yaml
apiVersion: v1     #必选，版本号，例如v1
kind: Pod         #必选，资源类型，例如 Pod
metadata:         #必选，元数据
  name: string     #必选，Pod名称
  namespace: string  #Pod所属的命名空间,默认为"default"
  labels:           #自定义标签列表
    - name: string                 
spec:  #必选，Pod中容器的详细定义
  containers:  #必选，Pod中容器列表
  - name: string   #必选，容器名称
    image: string  #必选，容器的镜像名称
    imagePullPolicy: [ Always|Never|IfNotPresent ]  #获取镜像的策略 
    command: [string]   #容器的启动命令列表，如不指定，使用打包时使用的启动命令
    args: [string]      #容器的启动命令参数列表
    workingDir: string  #容器的工作目录
    volumeMounts:       #挂载到容器内部的存储卷配置
    - name: string      #引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
      mountPath: string #存储卷在容器内mount的绝对路径，应少于512字符
      readOnly: boolean #是否为只读模式
    ports: #需要暴露的端口库号列表
    - name: string        #端口的名称
      containerPort: int  #容器需要监听的端口号
      hostPort: int       #容器所在主机需要监听的端口号，默认与Container相同
      protocol: string    #端口协议，支持TCP和UDP，默认TCP
    env:   #容器运行前需设置的环境变量列表
    - name: string  #环境变量名称
      value: string #环境变量的值
    resources: #资源限制和请求的设置
      limits:  #资源限制的设置
        cpu: string     #Cpu的限制，单位为core数，将用于docker run --cpu-shares参数
        memory: string  #内存限制，单位可以为Mib/Gib，将用于docker run --memory参数
      requests: #资源请求的设置
        cpu: string    #Cpu请求，容器启动的初始可用数量
        memory: string #内存请求,容器启动的初始可用数量
    lifecycle: #生命周期钩子
  postStart: #容器启动后立即执行此钩子,如果执行失败,会根据重启策略进行重启
  preStop: #容器终止前执行此钩子,无论结果如何,容器都会终止
    livenessProbe:  #对Pod内各容器健康检查的设置，当探测无响应几次后将自动重启该容器
      exec:         #对Pod容器内检查方式设置为exec方式
        command: [string]  #exec方式需要制定的命令或脚本
      httpGet:       #对Pod内个容器健康检查方法设置为HttpGet，需要制定Path、port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:     #对Pod内个容器健康检查方式设置为tcpSocket方式
         port: number
       initialDelaySeconds: 0       #容器启动完成后首次探测的时间，单位为秒
       timeoutSeconds: 0          #对容器健康检查探测等待响应的超时时间，单位秒，默认1秒
       periodSeconds: 0           #对容器监控检查的定期探测时间设置，单位秒，默认10秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
  restartPolicy: [Always | Never | OnFailure]  #Pod的重启策略
  nodeName: <string> #设置NodeName表示将该Pod调度到指定到名称的node节点上
  nodeSelector: obeject #设置NodeSelector表示将该Pod调度到包含这个label的node上
  imagePullSecrets: #Pull镜像时使用的secret名称，以key：secretkey格式指定
  - name: string
  hostNetwork: false   #是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
  volumes:   #在该pod上定义共享存储卷列表
  - name: string    #共享存储卷名称 （volumes类型有很多种）
    emptyDir: {}       #类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
    hostPath: string   #类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
      path: string                #Pod所在宿主机的目录，将被用于同期中mount的目录
    secret:          #类型为secret的存储卷，挂载集群与定义的secret对象到容器内部
      scretname: string  
      items:     
      - key: string
        path: string
    configMap:         #类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
      name: string
      items:
      - key: string
        path: string
```

### 3.1.2 查询子选项  
```shell
# 查询pod的yaml文件怎么写
kubectl explain 资源类型
kubectl explain 资源类型.属性

# 例子
kubectl explain pod
kubectl explain pod.metadata
```

## 3.2 详细配置  
### 3.2.1 containers

- 配置项  
```shell
# 使用这个命令可以得到containers的配置项
kubectl explain pod.spec.containers

# 如下
KIND:     Pod
VERSION:  v1
RESOURCE: containers <[]Object>   # 数组，代表可以有多个容器FIELDS:
  name  <string>     # 容器名称
  image <string>     # 容器需要的镜像地址
  imagePullPolicy  <string> # 镜像拉取策略 
  command  <[]string> # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
  args   <[]string> # 容器的启动命令需要的参数列表 
  env    <[]Object> # 容器环境变量的配置
  ports  <[]Object>  # 容器需要暴露的端口号列表
  resources <Object> # 资源限制和资源请求的设置
```

- 基本配置  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-base
  namespace: dev
  labels:
    user: demo
spec:
  containers: # 可以定义多个容器
    - name: nginx # 容器名字
      image: nginx:1.17.1 # 容器镜像
    - name: mysql
      image: mysql:5.7  
```

- imagePullPolicy 拉取镜像策略  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
    name: demo
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      imagePullPolicy: Always/IfNotPresent/Never

# Always：总是从仓库拉取
# IfNotPresent：本地有就用，没有就拉
# Never：只用本地，没有就报错

# 如果镜像tag为具体的版本号，默认策略是IfNotPresent。
# 如果镜像tag为latest（最终版本），默认策略是Always。
```

- command 启动命令  
可以让容器在启动后自动执行一个命令  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  namespace: demo
  labels:
    user: demo
spec:
  containers:
    - name: nginx
      image: ngingx:1.17.1
      imagePullPolicy: Never
    - name: busyBox
      image: busyBox:1.3
      # 容器启动后会执行下面的命令
      command: ["/bin.sh", "-c", "touch /temp/hello.txt;while true; do /bin/echo $(date +%T) >> /temp/hello.txt;sleep 3; done;"]
#######
# command：用于在Pod中的容器初始化完毕之后执行一个命令。
# 这里稍微解释下command中的命令的意思：
#  "/bin/sh","-c"：使用sh执行命令。
#  touch /tmp/hello.txt：创建一个/tmp/hello.txt的文件。
#  while true;do /bin/echo $(date +%T) >> /tmp/hello.txt;#  sleep 3;done：每隔3秒，向文件写入当前时间
#######

#######
# command已经可以完成启动命令和传递参数的功能，为什么还要提供一个args选项？
# kubernetes中的command和args两个参数其实是为了实现覆盖Dockerfile中的ENTRYPOINT的功能：
#
# ● 如果command和args均没有写，那么用Dockerfile的配置。
# ● 如果command写了，但是args没有写，那么Dockerfile默认的配置会被忽略，执行注入的command。
# ● 如果command没有写，但是args写了，那么Dockerfile中配置的ENTRYPOINT命令会被执行，使用当前args的参数。
# ● 如果command和args都写了，那么Dockerfile中的配置会被忽略，执行command并追加上args参数。
#
#######
```

- env 环境变量（不推荐）  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env
  namespace: dev
  labels: 
    user: envdemo
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      imagePullPollicy: ifNotPresent
      env:
        - name: "username"
          value: "wahaha"
        - name: "password"
          value: "pasiwode"

# 进入容器后可以用echo $username 来使用环境变量，但是这种做法不推荐
```

- ports 端口

```yaml
apiVersion: v1
kindL: Pod
metadata:
  name: port
  namespace: dev
  labels:
    user: wahaha
spec:
  containers:
    - name: ngingx
      image: nginx:1.17.1
      imagePullPolicy: Always
      # 自选项也是数组，
      ports:
        - name: nginx-port  # 端口名称，如果执行，必须保证name在Pod中是唯一的
          containerPort: 80 # 容器要监听的端口
          protocol: TCP # 端口协议
```

- resources 资源配额

容器运行需要占用资源，resources选项来制定容器需要的最大和最小的资源。  
limits：最大资源，容器超过时会被终止然后进行重启
requests：最小资源，如果资源不够则容器无法启动

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
  labels:
    user: wahaha
spec:
  containers:
    - name: nginx
      image: nginx:1.18.1
      imagePullPolicy: ifNotPresent
      resources:
        # 资源下限
        requests:
          cpu: "2"
          memory: "10Gi"
        # 资源上限
        limits:
          cpu: "4"
          memory: "24Gi"

# cpu：core数，可以为整数或小数。
# memory：内存大小，可以使用Gi、Mi、G、M等形式。

# GiB 和 GB的区别
#
# 1KB == 1,000 Byte
# 1MB == 1,000 KB
# 1GB == 1,000 MB

# 1KiB == 1,024 Byte
# 1MiB == 1,024 KiB
# 1GiB == 1,024 MiB
```