# 1 资源管理
## 1.1 资源管理方式
### 1.1.1 分类
- 命令式对象管理:直接使用命令去管理 
```shell
kubectl run nginx-pod --image=nginx:1.17.1 --port=80
```

- 命令是对象配置:通过命令配置和配置文件去操作k8s资源
```shell
kubectl create/patch -f nginx-pod.yaml
```

- 声明式对象配置:通过apply命令和配置文件去操作k8s资源，仅能更新和创建
```shell
kubectl apply -f nginx-pod.yaml
```

### 1.1.2 命令式对象管理
- kubectl命令  
  kubectl能够对集群本身进行管理，并且能够在集群上进行容器化的应用安装部署其语法如下
```shell
kubectl [command] [type] [name] [flags]
```

- command  
  指定要对资源执行的操作，比如create get delete

- type  
  指定资源的类型，比如deployment，pod，service

- name  
  指定资源的名称，名称大小写敏感

- flags  
  指定额外的可选参数

```shell
# 查看所有pod
kubectl get pod

# 查看某个pod
kubectl get pod pod_name

# 查看某个pod，以yaml格式展示结果
kubectl get pod pod_name -o yaml
```

- 案例  
创建dev名称空间 -> 创建一个pod -> 查看pod -> 删除pod -> 删除名称空间  
```shell
liangxiaole@ryoushous-MBP yaml % kubectl create ns dev
namespace/dev created


liangxiaole@ryoushous-MBP yaml % kubectl get ns
NAME              STATUS   AGE
default           Active   28m
dev               Active   7s
kube-node-lease   Active   28m
kube-public       Active   28m
kube-system       Active   28m


liangxiaole@ryoushous-MBP yaml % kubectl run pod --image=nginx:1.17.1 -n dev
pod/pod created


liangxiaole@ryoushous-MBP yaml % kubectl get pod -n dev
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          10s


liangxiaole@ryoushous-MBP yaml % kubectl delete pod pod -n dev 
pod "pod" deleted


liangxiaole@ryoushous-MBP yaml % kubectl get pod -n dev         
No resources found in dev namespace.


liangxiaole@ryoushous-MBP yaml % kubectl delete ns dev
namespace "dev" deleted


liangxiaole@ryoushous-MBP yaml % kubectl get ns       
NAME              STATUS   AGE
default           Active   33m
kube-node-lease   Active   33m
kube-public       Active   33m
kube-system       Active   33m
```

### 1.1.2 命令式对象配置  
- 案例  
创建dev名称空间 -> 创建一个pod -> 查看pod -> 删除pod -> 删除名称空间   
[yaml](./yaml/nginxpod-1.1.2.yaml) 

```yaml
apiVersion: v1
kind: Namespace
metadata: 
  name: dev


---


apiVersion: v1
kind: Pod
metadata: 
  name: nginxpod
  namespace: dev
spec: 
  containers:
  - name: nginx-containers
    image: nginx:1.17.2
```

```shell
liangxiaole@ryoushous-MBP yaml % kubectl create -f nginxpod-1.1.2.yaml 
namespace/dev created
pod/nginxpod created


liangxiaole@ryoushous-MBP yaml % kubectl get ns dev
NAME   STATUS   AGE
dev    Active   16s

liangxiaole@ryoushous-MBP yaml % kubectl get pod -n dev
NAME       READY   STATUS    RESTARTS   AGE
nginxpod   1/1     Running   0          51s


liangxiaole@ryoushous-MBP yaml % kubectl delete -f nginxpod-1.1.2.yaml 
namespace "dev" deleted
pod "nginxpod" deleted
```

### 1.1.3 声明式对象配置  
- 案例  
创建dev名称空间 -> 创建一个pod -> 查看pod  
如果再次执行apply，如果没有编程yaml的话，就什么都不执行。如果改变了的话就更新  
[yaml](./yaml/nginxpod-1.1.2.yaml) 

```yaml
apiVersion: v1
kind: Namespace
metadata: 
  name: dev


---


apiVersion: v1
kind: Pod
metadata: 
  name: nginxpod
  namespace: dev
spec: 
  containers:
  - name: nginx-containers
    image: nginx:1.17.2
```


```shell
liangxiaole@ryoushous-MBP yaml % kubectl apply -f nginxpod-1.1.2.yaml 
namespace/dev created
pod/nginxpod created


liangxiaole@ryoushous-MBP yaml % kubectl get ns 
NAME              STATUS   AGE
default           Active   52m
dev               Active   9s
kube-node-lease   Active   52m
kube-public       Active   52m
kube-system       Active   52m


liangxiaole@ryoushous-MBP yaml % kubectl get pod -n dev
NAME       READY   STATUS    RESTARTS   AGE
nginxpod   1/1     Running   0          17s


# 变更了配置文件再次执行，那么就会显示configured
liangxiaole@ryoushous-MBP yaml % kubectl apply -f nginxpod-1.1.2.yaml
namespace/dev unchanged
pod/nginxpod configured
```

# 2 实战
## 2.1 Namespace  
实现多套环境的资源隔离，或者多租户的资源隔离。在同一个ns中的pod中可以相互访问，不再同一个则不能相互访问。将不同的ns交给不同的租户管理，那么就实现了资源的隔离。  

### 2.1.1 查  
```shell
liangxiaole@ryoushous-MBP yaml % kubectl get ns default
NAME      STATUS   AGE
default   Active   66m


# 查看ns详情 describe
liangxiaole@ryoushous-MBP yaml % kubectl describe ns default
Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active # Active命名空间正在使用中，Terminating 正在删除

No resource quota.

No LimitRange resource.
# resource quota 针对ns做的资源限制
# LimitRange resource 针对ns中每个组件做的资源限制
```

### 2.1.2 增  
```shell
liangxiaole@ryoushous-MBP yaml % kubectl create ns dev
namespace/dev created
liangxiaole@ryoushous-MBP yaml % 
```

### 2.1.3 删除  
```shell
liangxiaole@ryoushous-MBP yaml % kubectl delete ns dev
namespace "dev" deleted
```

## 2.2 Pod  
Pod是K8s集群进行管理的最小单元，运行在容器中，容器存在于pod中。   
### 2.2.1 增  
- 命令行操作  
k8s没有提供单独运行pod的命令，都是通过pod控制器来实现的
```shell
# 命令： kubectl run pod控制器名称 【参数】
# --image 指定pod镜像
# --port 指定端口
# --namespace 指定ns

liangxiaole@ryoushous-MBP yaml % kubectl run nginx --image=nginx:1.17.1 --port=80 --namespace dev
pod/nginx created
```

- 配置操作  
[yaml](./yaml/nginxpod-2.2.1.yaml) 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
spec:
  containers:
  - image: nginx:1.17.1
    name: nginx
    ports:
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```
```shell
# create
liangxiaole@ryoushous-MBP yaml % kubectl create -f nginxpod-2.2.1.yaml
pod/nginx created

# delete
liangxiaole@ryoushous-MBP yaml % kubectl delete -f nginxpod-2.2.1.yaml 
pod "nginx" deleted
```

## 2.3 Label  
在资源上添加标识，对资源进行选择。  
> 一个label会议键值对的方式附加到各种对象上，比如Node Pod Service  
> 一个资源对象可以定义任意数量的Label，同一个Label也可以被添加到任意数量的资源对象上去  
> Label通常在资源对象定义时明确，当然也可以在对象创建后动态添加或者删除  

- 命令方式  
```shell  
# 打标签
liangxiaole@ryoushous-MBP yaml % kubectl label pod nginx -n dev version=1.0
pod/nginx labeled


# 查看标签
liangxiaole@ryoushous-MBP yaml % kubectl get pods -n dev --show-labels     
NAME    READY   STATUS    RESTARTS   AGE   LABELS
nginx   1/1     Running   0          87s   version=1.0


# 追加不同标签
liangxiaole@ryoushous-MBP yaml % kubectl label pod nginx -n dev env=back
pod/nginx labeled


# 查看不同标签
liangxiaole@ryoushous-MBP yaml % kubectl get pods -n dev --show-labels  
NAME    READY   STATUS    RESTARTS   AGE    LABELS
nginx   1/1     Running   0          113s   env=back,version=1.0


# 覆盖标签
liangxiaole@ryoushous-MBP yaml % kubectl label pod nginx -n dev env=front --overwrite
pod/nginx labeled


# 查看覆盖的标签
liangxiaole@ryoushous-MBP yaml % kubectl get pods -n dev --show-labels               
NAME    READY   STATUS    RESTARTS   AGE     LABELS
nginx   1/1     Running   0          2m39s   env=front,version=1.0


# 删除标签
liangxiaole@ryoushous-MBP yaml % kubectl label pod nginx -n dev env-
pod/nginx unlabeled


# 查看删除后的标签
liangxiaole@ryoushous-MBP yaml % kubectl get pods -n dev --show-labels               
NAME    READY   STATUS    RESTARTS   AGE    LABELS
nginx   1/1     Running   0          6m2s   version=1.0
```

- 配置方式  
```yaml
apiVersion: v1
kind: Pod
metadata: 
  name: nginx
  namespace: dev
  labels:
    version: "3.0"
    env: "test"
spec:
  containers:
  - name: nginx
    image: nginx:1.17.1
    ports: 
    - name: nginx-port
      containerPort: 80
      protocol: TCP
```

## 2.4 Deployment  
k8s中，pod是最小的控制单元，但是k8s很少直接控制pod，一般都是通过pod控制器来完成的。pod控制器用于pod的管理，确保pod资源符合预期的状态，当pod资源出现故障的时候，会尝试进行重启或者重建pod。 

- 命令操作  
```shell
# 新建deploy
liangxiaole@ryoushous-MBP yaml % kubectl create deployment nginx --image=nginx:1.17.1 --port=80 --replicas=3 --namespace=dev
deployment.apps/nginx created


# 查看deploy
liangxiaole@ryoushous-MBP yaml % kubectl get deployment -n dev --show-labels
NAME    READY   UP-TO-DATE   AVAILABLE   AGE    LABELS
nginx   3/3     3            3           100s   app=nginx


# 查看pod
liangxiaole@ryoushous-MBP yaml % kubectl get pod -n dev --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   LABELS
nginx-657fdb5865-g59hp   1/1     Running   0          77s   app=nginx,pod-template-hash=657fdb5865
nginx-657fdb5865-lr77k   1/1     Running   0          77s   app=nginx,pod-template-hash=657fdb5865
nginx-657fdb5865-w9sqz   1/1     Running   0          77s   app=nginx,pod-template-hash=657fdb5865


# 删除deploy
liangxiaole@ryoushous-MBP yaml % kubectl delete deployment nginx -n dev
deployment.apps "nginx" deleted
```

- 配置文件操作  
[yaml](./yaml/nginxdeploy-2.4.yaml) 
```yaml
# deployment
apiVersion: app/v1
kind: Deployment
metadata: 
  name: nginx
  namespace: dev
# deployment 的配置，包括标签选择和副本数量  
spec:
  replicas: 3
  selector:
    matchLabels:
      run: nginx
  # deployment管理的资源    
  template: 
    metadata:
      name: nginx
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx:1.17.1
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
```

```shell
# create
liangxiaole@ryoushous-MBP yaml % kubectl apply -f nginxdeploy-2.4.yaml
```

## 2.5 Service  
每个pod都会被单独分配一个IP，但是又两个问题    
> Pod IP会随着pod重建产生变化  
> 是内部IP，外部无法访问  

Service来解决这个问题，可以被看作是一组同类的pod对外的访问接口。借助service可以更好实现的实现服务的发现和负载均衡。  


### 2.5.1 集群内可访问的service（ClusterIP）  
```shell

# 创建deploy
liangxiaole@ryoushous-MBP yaml % kubectl create deploy nginx --image=nginx:1.17.1 --port=80 --replicas=3 -n dev 
deployment.apps/nginx created

# 创建并且暴露service
liangxiaole@ryoushous-MBP yaml % kubectl expose deploy nginx --name=svc-nginx1 --type=ClusterIP --port=80 --target-port=80 -n dev
service/svc-nginx1 exposed

# 查看service
liangxiaole@ryoushous-MBP yaml % kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
svc-nginx1   ClusterIP   10.106.158.38   <none>        80/TCP    20s
```

### 2.5.2 集群外可访问的service（NodePort）  
```shell

# 创建deploy
liangxiaole@ryoushous-MBP yaml % kubectl create deploy nginx --image=nginx:1.17.1 --port=80 --replicas=3 -n dev 
deployment.apps/nginx created

# 创建并且暴露service
liangxiaole@ryoushous-MBP yaml % kubectl expose deploy nginx --name=svc-nginx2 --type=NodePort --port=80 --target-port=80 -n dev
service/svc-nginx1 exposed

# 查看
liangxiaole@ryoushous-MBP yaml % kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-nginx1   ClusterIP   10.106.158.38   <none>        80/TCP         5m28s
svc-nginx2   NodePort    10.96.219.0     <none>        80:30963/TCP   8s
```

### 2.5.3 删除  
```shell
liangxiaole@ryoushous-MBP yaml % kubectl get svc -n dev
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-nginx1   ClusterIP   10.106.158.38   <none>        80/TCP         8m50s
svc-nginx2   NodePort    10.96.219.0     <none>        80:30963/TCP   3m30s

# delete 
liangxiaole@ryoushous-MBP yaml % kubectl delete svc svc-nginx1 -n dev
service "svc-nginx1" deleted

# search
liangxiaole@ryoushous-MBP yaml % kubectl get svc -n dev              
NAME         TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
svc-nginx2   NodePort   10.96.219.0   <none>        80:30963/TCP   3m44s
```


### 2.5.4 配置文件  

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nginx
  namespace: dev
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: nginx
  type: ClusterIP
```

```shell
# create
liangxiaole@ryoushous-MBP yaml % kubectl apply -f svc-2.5.4.yaml
service/svc-nginx created

# search
liangxiaole@ryoushous-MBP yaml % kubectl get svc -n dev
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
svc-nginx   ClusterIP   10.107.216.33   <none>        80/TCP    10s
```

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

## 3.3 Pod生命周期

### 3.3.1 概述  
pod从创建到销毁的时间段是pod生命周期  
- pod创建过程
- 运行初始化容器过程（init container）
- 运行主容器（main container）  
        - 容器启动后钩子（post start），容器终止前钩子（pre stop）
        - 容器存活探测（liveness probe），就绪性探测（readiness probe）
- pod终止过程  

● 在整个生命周期中，Pod会出现5种状态（相位），分别如下：  
- 挂起（Pending）：API Server已经创建了Pod资源对象，但它尚未被调度完成或者仍处于下载镜像的过程中。  
- 运行中（Running）：Pod已经被调度到某节点，并且所有容器都已经被kubelet创建完成。  
- 成功（Succeeded）：Pod中的所有容器都已经成功终止并且不会被重启。  
- 失败（Failed）：所有容器都已经终止，但至少有一个容器终止失败，即容器返回了非0值的退出状态。  
- 未知（Unknown）：API Server无法正常获取到Pod对象的状态信息，通常由于网络通信失败所导致。  

### 3.3.2 初始化容器  
会在主容器之前进行启动，有启动的先后顺序，如果卡住了则后面的容器无法启动。

下面的案例，有两个初始化容器，创建的时候会执行命令，但是这个命令是走不通的，所以k8s在创建的时候就会卡住
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-container
  nemaspace: dev
  labels:
    name: wahaha
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      imagePullPolicy: ifNotPresent
      ports:
        - name: nging-port
          containerPort: 80
          protocol: TCP
      resources:
        limits:
          cpu: "2"
          memory: "10Gi"
        requests:
          cpu: "2"
          memory: "10Gi"
  # 初始化容器配置
  initContainers:
    - name: test-mysql
      image: busybox:1.30
      command: ["sh", "-c", "until ping 1.1.1.1 -c 1; do echo waiting for mysql ...; sleep 2; done"]
      securityContext:
        privileged: true # 使用特权模式运行容器
    - name: "test-redis"
      image: busybox:1.30
      command: ["sh","-c","until ping 192.168.18.104 -c 1;do echo waiting for redis ...;sleep 2;done;"]
```

### 3.3.3 钩子函数  
定义  
- post start 容器创建之后执行，如果失败则重启容器
- pre stop 容器终止之前执行，完成之后终止容器，完成之前会阻塞删除容器的操作  

支持三种动作  
- exec命令：在容器内执行一次命令
```yaml
lifecycle:
  postStart:
    exec:
      command:
        - cat
        - /tmp/healthy
```

- topSocket：在当前容器尝试访问socket
```yaml
lifecycle:
  postSttart:
    tcpSocket:
      port: 8080
```

- httpGet：在当前容器向某url发起http请求  
```yaml
lifecycle:
  postStart:
    httpGet:
      path: /URL
      port: 80
      host: 主机地址
      scheme: HTTP/HTTPS
```

- 案例：容器启动后修改html内容，终止前停掉nginx服务  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ngingx-life
  nemaspace: dev
  labels:
    name: wahaha
spec:
  containers:
    - name: nginx-pod
      image: ngingx:1.17.1
      imagePullPolicy: Never
      ports:
        - name: nginx-port
          containerPort: 80
          protocol: TCP
      resources:
        limits:
          cpu: "4"
          memory: "5Gi"
        requests:
          cpu: "8"
          memory: "10Gi"
      # 容器生命周期配置
      lifecycle:
        postStart:  # 启动后，修改掉nginx主页的显示内容
          exec:
            command:  ["/bin/sh","-c","echo postStart ... > /usr/share/nginx/html/index.html"]
        preStop:  # 中之前，停掉nginx的服务
          exec:
            command: ["/usr/sbin/nginx","-s","quit"]
```

### 3.3.4 容器探测  

kubernetes提供了两种探针来实现容器探测，分别是  
- livenessProbe: 存活性探测，决定是否重启容器。
- readinessProbe：就绪性探测，决定是否将请求转发给容器。  

支持三种动作  
- exec命令：在容器内执行一次命令，如果命令执行的退出码为0，则认为程序正常，否则不正常。
```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
```

- tcpSocket: 访问一个容器的端口，能建立链接为正常，否则不正常
```yaml
livenessProbe:
  tcpSocket:
    port: 8080
```

- httpGet: 访问一个url，返回状态码在200-399位正常，否则不正常
```yaml
livenessProbe:
  httpGet:
    path: /URL
    port: 80
    host: 主机名
    scheme: http/https # 协议名字
```

### 3.3.4 重启策略  
容器探测中，一旦出现问题，k8s就会重启容器。有三种策略可以选择  

***restartPolicy***
- Always：默认值，总是重启
- OnFailure：容器终止运行且推出码不为0时重启
- Never：不论状态如何，都不重启该容器  

案例
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-restart
  namespace: dev
  labels:
    name: wahaha
spec:
  containers:
    - name: nginx-wahaa
      image: nginx:1.18.1
      imagePullPolicy: Always
      ports:
        - name: wahaha-port
          containerPort: 80
          protocol: TCP
      livenessProbe:
        httpGet:
          path: /url
          port: 80
          host: 1.0.0.1
          scheme: https
  # 重启策略  
  restartPolicy: Never
```

### 3.3.4 Pod调度  
即控制pod部署到哪个节点上面  
- 自动调度：放在哪个node上完全由scheduler决定
- 定向调度：NodeName，NodeSelector
- 亲和性调度：NodeAffinity，PodAffinity，PodAntiAffinity
- 污点（容忍）调度：Taints，Toleration

***定向调度***  
在pod文件上声明要调度到的nodeName或者nodeSelector，就会往上面调。而且是强制的，如果node或者选择器不存在，也会调，但是会失败。

> nodeName
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wahaha
  namespace: wahaha-dev
  labels:
    name: wahaha
spec:
  containers:
    - name: nginx
      image: ngins:1.171.
      imagePullPolicy: Always
      ports:
        - name: nginx-name
          containerPort: 80
          protocol: TCP
  nodeName: worker1 # 指定调度到worker1节点上面
```

> nodeSelector    

调度到加了标签的node上面去    
给worker1和worker2节点分别打上标签  
```shell
kubectl label node worker1 nodeenv=pro
kubectl label node worker2 nodeenv=test
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wahaha
  namespace: wahaha-dev
  labels:
    name: wahaha
spec:
  containers:
    - name: nginx
      image: ngins:1.171.
      imagePullPolicy: Always
      ports:
        - name: nginx-name
          containerPort: 80
          protocol: TCP
  nodeSelector: # 指定调度到打了下面标签的节点上面
    nodeenv: pro
```

***亲和性调度***  
优先选择满足条件的节点进行调度，不满足条件的话，也可以选择别的节点进行调度。  

>   nodeAffinity（node亲和性）：以Node为目标，解决Pod可以调度到那些Node的问题。  
>   podAffinity（pod亲和性）：以Pod为目标，解决Pod可以和那些已存在的Pod部署在同一个节点中的问题。  
>   podAntiAffinity（pod反亲和性）：以Pod为目标，解决Pod不能和那些已经存在的Pod部署在同一节点中的问题。

> 关于亲和性和反亲和性的使用场景的说明：  
● 亲和性：如果两个应用频繁交互，那么就有必要利用亲和性让两个应用尽可能的靠近，这样可以较少因网络通信而带来的性能损耗。  
● 反亲和性：当应用采用多副本部署的时候，那么就有必要利用反亲和性让各个应用实例打散分布在各个Node上，这样可以提高服务的高可用性。

___

> nodeAffinity  

```yaml
pod.spec.affinity.nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:  #Node节点必须满足指定的所有规则才可以，相当于硬限制
    nodeSelectorTerms: # 节点选择列表
      matchFields: # 按节点字段列出的节点选择器要求列表  
      matchExpressions: # 按节点标签列出的节点选择器要求列表(推荐)
        key:   # 键
        values:  # 值
        operator: # 关系符 支持Exists, DoesNotExist, In, NotIn, Gt, Lt

  preferredDuringSchedulingIgnoredDuringExecution:  #优先调度到满足指定的规则的Node，相当于软限制 (倾向)     
    preference:  # 一个节点选择器项，与相应的权重相关联
      matchFields: # 按节点字段列出的节点选择器要求列表
      matchExpressions:  # 按节点标签列出的节点选择器要求列表(推荐)
        key: # 键
        values: # 值
        operator: # 关系符 支持In, NotIn, Exists, DoesNotExist, Gt, Lt  
    weight: # 倾向权重，在范围1-100。
```

用法说明
```yaml
- matchExpressions:
  # 节点上，有标签的key是nodeenv
  - key: nodeenv 
    operator: Exists
  # 节点上，标签的key是nodeenv，并且值是["xxx", "yyy"]其中之一
  - key: nodeenv 
    operator: In
      values: ["xxx", "yyy"]
  # 节点上，标签的key是nodeenv，并且值 > 650
  - key: nodeenv
    operator: Gt
      value: "650"
```

强制亲和案例  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: required
  namespace: dev
  labels:
    name: wahaha
spec:
  containers:
    - name: required
      image: nginx:1.17.1
      imagePullPolicy: ifNotPresent
      ports:
        - name: required-name
          containerPort: 80
          protocol: TCP
  # 强制亲和配置
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: nodeenv
              operator: In
              value:
                - "xxx"
                - "yyy"

```

偏向亲和案例  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ngingx
  namespace: dev
  labels:
    user: wahaha
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      imagePullPolicy: Always
      ports:
        - name: nginx-name
          containerPort: 80
          protocol: UDP
  # 偏向亲和配置
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - preference:
            - matchExpressions:
                - key: nodeenv
                  operator: In
                  values:
                    - "xxx"
                    - "yyy"
          weight: 1
```

> podAffinity  

> topologyKey用于指定调度的作用域，例如:
>>● 如果指定为kubernetes.io/hostname，那就是以Node节点为区分范围。  
>> ● 如果指定为beta.kubernetes.io/os，则以Node节点的操作系统类型来区分。

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
    - name: "nginx-wahaha"
      image: nginx:1.17.1
      imagePullPolicy: Never
      ports:
        - name: port-name
          containerPort: 80
          protocol: TCP
  affinity:
    podAffinity:
      # 硬限制
      # 该Pod必须和拥有标签podenv=xxx或者podenv=yyy的Pod在同一个Node上
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: podenv
                operator: In
                values:
                  - "xxx"
                  - "yyy"
          topologyKey: kubernetes.io/hostname

```

> podAntiAffinity  

让新创建的Pod和参照的Pod不在一个区域的功能  
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
    - name: "nginx-wahaha"
      image: nginx:1.17.1
      imagePullPolicy: Never
      ports:
        - name: port-name
          containerPort: 80
          protocol: TCP
  affinity:
    antiPodAffinity: # Pod 反亲和
      # 硬限制
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: podenv
                operator: In
                values:
                  - "xxx"
                  - "yyy"
          topologyKey: kubernetes.io/hostname
```

### 3.3.6 污点和容忍  

 - 污点  

给node打上污点，拒绝pod部署进来，甚至可以把已有的node驱逐出去。
命令如下。
```shell
# effect 有三种选择
# PreferNoSchedule: 尽量不往这个node部署，除非没有其他node
# NoSchedule: 不会部署新的pod在这个node，但不影响旧的node
# NoExecute: 不会部署新的，且要驱逐旧的

# 命令
kubectl taint node xxx key=value:effect
```

一些命令和例子  
```shell
# 设置污点  
kubectl taint node xxx tag=wahaha:PreferNoSchedule

# 去除污点
kubectl taint node xxx tag=wahaha-

# 去除所有污点
kubectl taint node xxx tag-
```

- 容忍  

想让一个pod部署到有污点的node，可以用容忍toleration来配置  
> kubectl explain pod.spec.tolerations  
......  
>>FIELDS:  
  key       # 对应着要容忍的污点的键，空意味着匹配所有的键  
  value     # 对应着要容忍的污点的值  
  operator  # key-value的运算符，支持Equal和Exists（默认）  
  effect    # 对应污点的effect，空意味着匹配所有影响  
  tolerationSeconds   # 容忍时间, 当effect为NoExecute时生  效，表示pod在Node上的停留时间

___
> 当operator为Equal的时候，如果Node节点有多个Taint，那么Pod每个Taint都需要容忍才能部署上去。  
>> 当operator为Exists的时候，有如下的三种写法：  
● 容忍指定的污点，污点带有指定的effect：  
● 容忍指定的污点，不考虑具体的effect：  
● 容忍一切污点（慎用）：  

```yaml
# ● 容忍指定的污点，污点带有指定的effect：  
tolerations:
  - key: "tag"
    operator: Exists
    effect: NoExecute # 添加容忍的规则，这里必须和标记的污点规则相同
```

```yaml
# ● 容忍指定的污点，不考虑具体的effect：  
tolerations:
  - key: "tag"
    operator: Exists
```

```yaml
# ● 容忍一切污点（慎用）： 
tolerations:
  - key: "tag"
```

案例  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration
  namespace: dev
spec:
  containers:
    - name: nginx-pod
      image: nginx:1.171.
      imagePullPolicy: ifNotPresent
      ports:
        - name: port-name
          containerPort: 80
          protocol: TCP
  # 容忍
  tolerations: 
    - key: "key"
      value: "wahaha"
      operator: Equal
      effect: NoExecute # 添加容忍的规则，必须和标记的污点规则相同
```

### 3.3.7 Qos  
- Guaranteed  

```yaml
# Pod 中的每个容器，包含初始化容器，必须指定内存请求和内存限制，并且两者要相等。
# Pod 中的每个容器，包含初始化容器，必须指定 CPU 请求和 CPU 限制，并且两者要相等。
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```

- Burstable  
```yaml
# Pod 不符合 Guaranteed QoS 类的标准。
# Pod 中至少一个容器具有内存或 CPU 请求，但是值不相等。
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-2
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-2-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```

- BestEffort  
```yaml
# Pod 中的容器必须没有设置内存和 CPU 限制或请求
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-3
  namespace: qos-example
spec:
  containers:
  - name: qos-demo-3-ctr
    image: nginx
```

- 应用  

一旦出现OOM，kubernetes为了保证服务的可用，会先删除QoS为BestEffort的Pod，然后删除QoS为Burstable的Pod，最后删除QoS为Guaranteed 的Pod。

# 4 Pod控制器  
## 4.1 介绍  

常见控制器  

- ReplicationController：比较原始的Pod控制器，已经被废弃，由ReplicaSet替代。
- ReplicaSet：保证指定数量的Pod运行，并支持Pod数量变更，镜像版本变更。
- Deployment：通过控制ReplicaSet来控制Pod，并支持滚动升级、版本回退。
- Horizontal Pod Autoscaler：可以根据集群负载自动调整Pod的数量，实现削峰填谷。
- DaemonSet：在集群中的指定Node上都运行一个副本，一般用于守护进程类的任务。
- Job：它创建出来的Pod只要完成任务就立即退出，用于执行一次性任务。
- CronJob：它创建的Pod会周期性的执行，用于执行周期性的任务。
- StatefulSet：管理有状态的应用。

## 4.2 ReplicaSet （RS）
主要作用是保证一定数量的pod能正常运行，如果故障了，就会控制pod重启  

- 资源文件  
```yaml
apiVersion: app
kind: ReplicaSet
metadata:
  name: demo
  namespace: dev
  labels: # 这个是控制器本身的标签，不是标签选择器
    controller: replicaSet
spec:
  replicas: 3 # pod的数量，副本数量
  selector: # 选择器，这个才是真正用来选择标签的
    matchLabels:
      app: nginx-pod
    matchExpressions:
      - {key: app, operator: In, values: ["xxx", "yyy"]}
template: # 这个就是pod的配置了,因为已经是pod了所以kind就不用写，直接metadata开始
  metadata:
    labels:
      app: nginx-pod
  spec:
    containers:
      - name: nginx
        image: nginx:1.17.1
        imagePullPolicy: Always
        ports:
          - name: nginx-port
            containerPort: 80
            protocol: TCP
```

- 扩缩容  

***修改manifest文件***
```shell
kubectl edit rs pc-replicaset -n dev

# 然后修改replica的数量，那么就会自动增加扩缩容了
```

***命令行***
```shell
kubectl scale rs rs pc-replicaset --replicas=6 -n dev
```

## 4.3 Deployment (Deploy)  

deploy并不直接管理pod，而是通过replicaSet来间接管理Pod.  
功能有：rs的所有功能，发布的停止与继续，版本滚动更新和版本回退

资源清单  

```yaml
apiVersion: apps/v1 # 版本号 
kind: Deployment # 类型 
metadata: # 元数据 
  name: # rs名称 
  namespace: # 所属命名空间 
  labels: #标签 
    controller: deploy 
spec: # 详情描述 
  replicas: 3 # 副本数量 
  revisionHistoryLimit: 3 # 保留历史版本，默认为10 
  paused: false # 暂停部署，默认是false 
  progressDeadlineSeconds: 600 # 部署超时时间（s），默认是600 
  strategy: # 策略 
    type: RollingUpdate # 滚动更新策略 
    rollingUpdate: # 滚动更新 
      maxSurge: 30% # 最大额外可以存在的副本数，可以为百分比，也可以为整数 maxUnavailable: 30% # 最大不可用状态的    Pod 的最大值，可以为百分比，也可以为整数 
  selector: # 选择器，通过它指定该控制器管理哪些pod 
    matchLabels: # Labels匹配规则 
      app: nginx-pod 
    matchExpressions: # Expressions匹配规则 
      - {key: app, operator: In, values: [nginx-pod]} 
  template: # 模板，当副本数量不足时，会根据下面的模板创建pod副本 
    metadata: 
      labels: 
        app: nginx-pod 
    spec: 
      containers: 
      - name: nginx 
        image: nginx:1.17.1 
        ports: 
        - containerPort: 80
```

### 4.3.1 创建deployment  
```yaml
apiVersion: app/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: dev
spec:
  replicas: 3 # 副本数量
  selector:   # 选择器
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx-pod
          image: nginx:1.17.1
          imagePullPolicy: Always
          ports:
            - containerPort: 80
```

### 4.3.2 扩缩容  

```shell
kubectl scale deploy nginx-deploy --replicas=5 -n dev
```

### 4.3.3 镜像更新  
- 重建更新  
在创建出新的Pod之前会先杀掉所有已经存在的Pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pc-deployment
  namespace: dev
  labels:
    app: deploy
template:
  replicas: 3
  strategy: # 镜像更新策略
    type: Recreate # 在创建出新的镜像之前会杀掉旧的
  selector:
    matchLabels:
      app: deploy
  spec: ... # pod 那一套
```

更新命令  
```shell
kubectl apply -f pc-deployment.yaml
```

镜像升级  
```shell
kubectl set image deployment pc-deployment nginx=nginx:1.17.2 -n dev
```

升级过程  
```yaml
kubectl get pod -n dev
```

- 滚动更新  

> RollingUpdate：滚动更新，就是杀死一部分，就启动一部分，在更新过程中，存在两个版本的Pod  
rollingUpdate：当type为RollingUpdate的时候生效，用于为rollingUpdate设置参数，支持两个属性：  
>>maxUnavailable：用来指定在升级过程中不可用的Pod的最大数量，默认为25%。  
maxSurge： 用来指定在升级过程中可以超过期望的Pod的最大数量，默认为25%。

```yaml
apiVersion: apps/v1
kind: Deployment
demtadata:
  name: pc-deploy
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-dev
  strategy:
    type: RollingUpdate # 镜像滚动升级
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  template:
    ... # pod
```

### 4.3.4 版本回退  

deploy支持升级过程中暂停，继续以及会滚等多种功能  
```shell
# 相关功能  
kubectl rollout 参数 deploy xx # 支持以下的选择  
# status 显示当前升级状态
# history  显示升级历史记录
# pause 暂停版本升级
# resume 继续暂停的升级
# restart 重启升级过程
# undo 会滚到上一个版本（可以使用 --to-revision会滚到指定版本）
```

- 查看状态  
```shell
kubectl rollout status deployment pc-deployment -n dev
```

- 查看升级历史记录
```shell
kubectl rollout history deployment pc-deployment -n dev
```

- 版本回退  
```shell
# 可以使用-to-revision=1回退到1版本，如果省略这个选项，就是回退到上个版本，即2版本
kubectl rollout undo deployment pc-deployment --to-revision=1 -n dev
```

## 4.4 DaemonSet(DS)

### 4.4.1 概述  
保证集群中的每一台或者指定街店上都运行一个副本，一般用于日志收集，节点监控等场景。  

- 资源清单  

```yaml
apiVersion: v1
kind: DaemonSet
metadata:
  name: ds
  namespaces: dev
  labels:
    controller: ds
spec:
  revisionHistoryLimit: 3 # 保留历史版本
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1 # 最大不可用状态的pod的最大值，可为百分比或者正数
  selector:
    matchLabels:
      app: nginx-pod
    matchExpressions:
      - key: app
        operator: In
        valus:
          - nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          imagePullPolicy: Always
          ports:
            - port: 80
```

### 4.4.2 创建  
```yaml
apiVersion: app
kind: DaemonSet
metadata:
  name: pc-daemonset
  namespace: dev
spec:
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadate:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - port: 80
```

```shell
kubectl create -f pd-daemonset.yaml
```

### 4.4.3 查看  
```shell
kubectl get ds -n dev -o wide
```

### 4.4.4 删除  
```shell
kubectl delete ds pd-daemonset.yaml -n dev
```

## 4.5 Job
### 4.5.1 概述
主要负责批量处理短暂的一次性任务。  
特点：1 当job创建的pod执行成功后，job回记录成功结束的pod的数量 2 成功结束的pod达到一定数量的同时，job将完成执行。

- 资源清单  
```yaml
apiVersion: v1
kind: Job
metadata:
  name: job
  namespace: dev
  labels:
    controller: job
spec:
  completions: 1 # 指定job需要成功运行pod的总次数，默认1
  parallelism: 1 # 自定job在同一时刻应该并发运行的数量，默认1
  activeDeadlineSeconds: 30 # job运行最大时长，超过的话k8s会终止
  backoffLimit: 6 # job失败后重试次数 默认6
  manualSelector: true # 是否可以使用selector选择器选择pod，默认false
  selector:
    matchLabels:
      app: counter-pod
    matchExpressions:
      - key: app
        operator: In
        values:
          - counter-pod
  template: 
    metadata:
      labels:
        app: counter-pod
    spec:
      restartPolicy: Never # 重启策略只能是Never或者OnFailure
      containers:
        - name: counter
          image: busybox:1.30
          command: ["/bin/sh", "-c" "....."]
```

> 关于模板中的重启策略的说明：  
>> ● 如果设置为OnFailure，则Job会在Pod出现故障的时候重启容器，而不是创建Pod，failed次数不变。  
● 如果设置为Never，则Job会在Pod出现故障的时候创建新的Pod，并且故障Pod不会消失，也不会重启，failed次数+1。  
● 如果指定为Always的话，就意味着一直重启，意味着Pod任务会重复执行，这和Job的定义冲突，所以不能设置为Always。


### 4.5.2 创建Job
```yaml
apiVersion: v1
kind: Job
metadata:
  name: pc-job
  namespace: dev
spec:
  manualSelector: true
  selector:
    matchLabels:
      app: counter-pod
  template:
    metadata:
      labels:
        app: counter-pod
    spec:
      restartPolicy: Never
      containers:
        - name: counter
          image: busybox:1.30
          command: ["..."]
```

```shell
kubectl create -f pc-job.yaml
```

### 4.5.3 查看Job  
```shell
kubectl get job -n dev -w
```

### 4.5.4 删除Job  
```shell
kubectl delete -f pc-job.yaml
```


## 4.6 CronJob(CJ)

### 4.6.1 概述  
CronJob可以在特定的时间点反复去执行job任务  

- 资源清单  
```yaml
apiVersion: batch
kind: CronJob
metadata:
  name: name
  namespace: dev
  labels:
    controller: cron-job
spec:
  schedule: 0 0 0 0 0 * # cron表达式
  concurrencyPollicy: # 并发执行策略
  failedJobsHistoryLimit: # 失败任务的历史记录数 默认1
  successfulJobsHistoryLimit: # 默认3
  jobTemplate: # Job控制器模版，用于位cronJob控制器生成job对象
    metadata: {}
    spec:
      completions: 1
      parallelism: 1
      activeDeadlineSeconds: 30
      backoffLimit: 6
      template:
        spec:
          restartPolicy: Never
          containers:
            - name: counter
              image: busybox:1.30
              command: ["..."]
```

> schedule：cron表达式，用于指定任务的执行时间。  
● */1  *  *  *  *：表示分钟  小时  日  月份  星期。  
● 分钟的值从0到59。  
● 小时的值从0到23。  
● 日的值从1到31。  
● 月的值从1到12。  
● 星期的值从0到6，0表示星期日。  
● 多个时间可以用逗号隔开，范围可以用连字符给出：* 可以作为通配符，/表示每...  

> concurrencyPolicy：并发执行策略  
● Allow：运行Job并发运行（默认）。  
● Forbid：禁止并发运行，如果上一次运行尚未完成，则跳过下一次运行。  
● Replace：替换，取消当前正在运行的作业并使用新作业替换它。  

### 4.6.2 创建CronJob  
```yaml
apiVersion: batch
kind: CronJob
metadata:
  name: pc-cronjob
  namespace: dev
spec:
  schedule: "*/1 * * * * "
  jobTemplate:
    metadata: {}
    spec:
      template: 
        spec:
          restartPolicy: Never
          containers:
            - name: counter
              image: busybox:1.17
              command: ["..."]
```

### 4.6.3 查看CronJob  
```shell
kubectl get cronjob -n dev -w
```

### 4.6.4 删除CronJob  
```shell
kubectl delete -f pc-fronjob.yaml
```

## 4.7 StatefulSet(有状态)  
### 4.7.1 概述  
- 无状态应用  
  - 认为pod都一样
  - 没有顺序要求
  - 不用考虑在哪个node上执行
  - 随意进行伸缩和扩展
- 有状态应用
  - 有顺序要求
  - 认为每个pod都是不一样的
  - 需要考虑在哪个Node上运行
  - 需要按照顺序进行伸缩扩展
  - 让每个pod都是独立的，保持pod启动顺序和唯一性
- StatefulSet是Kubernetes提供的管理有状态应用的负载管理控制器。
- StatefulSet部署需要HeadLinessService（无头服务）。
- StatefulSet常用来部署RabbitMQ集群、Zookeeper集群、MySQL集群、Eureka集群等。

> 为什么需要HeadLinessService（无头服务）？  
● 在用Deployment时，每一个Pod名称是没有顺序的，是随机字符串，因此是Pod名称是无序的，但是在StatefulSet中要求必须是有序 ，每一个Pod不能被随意取代，Pod重建后pod名称还是一样的。  
● 而Pod IP是变化的，所以是以Pod名称来识别。Pod名称是Pod唯一性的标识符，必须持久稳定有效。这时候要用到无头服务，它可以给每个Pod一个唯一的名称 。

### 4.7.2 创建StatefulSet  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-headliness
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: Node # 经clusterIP设置为Node，即可创建headliness Service
  type: ClusterIp
  ports:
    - port: 80 # Service 端口
      targetPort: 80 # Pod 端口
---
apiVersion: v1
kind: StatefuleSet
metadata:
  name: pc-statefuleSet
  namespace: dev
spec:
  replicas: 3
  serviceName: service-headliness
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      laebels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.18.1
          ports:
            - containerPort: 80
```

```shell
kubectl create -f pc-stateful.yaml
```

### 4.7.3 查看StatefulSet  
```shell
kubectl get statefulSet pc-startfulset -n dev -o wide
```

### 4.7.4 删除  
```shell
kubectl delete -f pc-stateful.yaml
```

# 5 Service  
## 5.1 service介绍  
- 在kubernetes中，Pod是应用程序的载体，我们可以通过Pod的IP来访问应用程序，但是Pod的IP地址不是固定的，这就意味着不方便直接采用Pod的IP对服务进行访问。

- 为了解决这个问题，kubernetes提供了Service资源，Service会对提供同一个服务的多个Pod进行聚合，并且提供一个统一的入口地址，通过访问Service的入口地址就能访问到后面的Pod服务。

- Service在很多情况下只是一个概念，真正起作用的其实是kube-proxy服务进程，每个Node节点上都运行了一个kube-proxy的服务进程。当创建Service的时候会通过API Server向etcd写入创建的Service的信息，而kube-proxy会基于监听的机制发现这种Service的变化，然后它会将最新的Service信息转换为对应的访问规则。
service给所有pod/控制器提供访问的入口  

## 5.2 service类型  
资源清单  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-name
  namespace: dev
spec:
  selector:
    app: nginx
  type: NodePort
  clusterIP: # 虚拟服务的IP地址
  sessionAffinity: # session亲和性，支持ClientIP、None两个选项，默认值为None
  ports: # 端口信息
    - port: 8080 # Service端口
      protocol: TCP # 协议
      targetPort : # Pod端口
      nodePort:  # 主机端口
```

> spec.type的说明：  
● ClusterIP：默认值，它是kubernetes系统自动分配的虚拟IP，只能在集群内部访问。  
● NodePort：将Service通过指定的Node上的端口暴露给外部，通过此方法，就可以在集群外部访问服务。  
● LoadBalancer：使用外接负载均衡器完成到服务的负载分发，注意此模式需要外部云环境的支持。  
● ExternalName：把集群外部的服务引入集群内部，直接使用。  

## 5.3 service使用  

### 5.3.1 环境准备

- 先创建3个pod，标签是app=nginx-pod
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pc-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: nginx:1.17.1
          ports:
            - containerPort: 80
```

### 5.3.2 ClusterIP类型  
- 创建service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-clusterip
  namespace: dev
spec:
  selector: 
    app: nginx-pod
  clusterIP: 10.10.10.10 # service地址，不写会默认生成一个
  type: ClusterIP
  ports:
    - port: 80 # service 端口
      targetPort: 80 # pod端口
```

```shell
kubectl create -f service-clusterip.yaml
```

- 查看service
```shell
kubectl get svc -n dev -o wide
```

- 查看详细信息  
```shell
kubectl describe svc service-clusterip -n dev
# 打印出来的信息里面，有个EndPoints字段，就是pod的入口
```

- 查看ipvs映射规则  
就是查看哪个ip转到哪个ip

```shell
ipvsadm -Ln
```

- 删除service
```shell
kubectl delete -f service-clusterip.yaml
```

### 5.3.3 HeadLiness类型  
- 概述  
在某些场景中，开发人员可能不想使用Service提供的负载均衡功能，而希望自己来控制负载均衡策略，针对这种情况，kubernetes提供了HeadLinesss Service，这类Service不会分配Cluster IP，如果想要访问Service，只能通过Service的域名进行查询。

- 创建  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-headliness
  namespace: dev
spec:
  selector:
    app: nginx-pod
    clusterIp: None # 将clusterIP设置为None，就可以创建headliness service
    type: ClusterIP
    ports:
      - port: 80
        targetPort: 80
```

```shell
kubectl create -f service-headliness.yaml
```

- 查看
```shell
kubectl describe svc service-headliness -n dev
```


- 查看详情
```shell
kubectl describe svc service-headliness -n dev
```

- 查看域名解析情况  
```shell
# 查看pod
kubectl get pod -n dev

# 进入pod中
kubectl exec -it pod名字 -n dev /bin/sh

# 查看配置文件
cat /ect/resolv.conf
```

### 5.3.4 NodePort类型  

- 概述  
在之前的案例中，创建的Service的IP地址只能在集群内部才可以访问，如果希望Service暴露给集群外部使用，那么就需要使用到另外一种类型的Service，称为NodePort类型的Service。NodePort的工作原理就是将Service的端口映射到Node的一个端口上，然后就可以通过NodeIP:NodePort来访问Service了。

- 创建
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nodeport
  namespace: dev
spec:
  selector:
    app: nginx-pod
  type: NodePort # 声明类型为NodePort
  ports:
    - port: 80 # Service端口
      targetPort: 80 # pod端口
      nodePort: 30002 # node端口
```

```shell
kubectl create -f service-nodeport.yaml
```

- 查看  
```shell
kubectl get svc service-nodeport -n dev -o wide
```

- 访问 

```shell
# 通过url:30002/ 即可访问对应的pod
```

### 5.3.5 LoadBalancer类型  
LoadBalancer和NodePort很相似，目的都是向外部暴露一个端口，区别在于LoadBalancer会在集群的外部再来做一个负载均衡设备，而这个设备需要外部环境的支持，外部服务发送到这个设备上的请求，会被设备负载之后转发到集群中。  
　　　nginx  
 　 　　/　　\  
 nodePort1  nodePort2  
　　　|　　　　|  
 pod pod　　pod  pod 


### 5.3.6 ExternalName类型  

- 概述  
ExternalName类型的Service用于引入集群外部的服务，它通过externalName属性指定一个服务的地址，然后在集群内部访问此Service就可以访问到外部的服务了。

- 创建  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-externalname
  namespace: dev
spec:
  type: ExternalName # Service类型是ExternalName
  ExternalName: www.google.com
```

```shell
kubectl create -f service-externalname.yaml
```

## 5.4 Ingress  

Service暴露服务有两种方式：NodePort和LoadBalancer，但是他们有一定缺点  
- nodePort方式会占用很多集群机器端口
- LoadBalancer 每个service都需要一个LB，且需要k8s之外的设备的支持

基于这种现状，kubernetes提供了Ingress资源对象，Ingress只需要一个NodePort或者一个LB就可以满足暴露多个Service的需求。作用相当于一个nginx  

> ● 可以理解为Ingress里面建立了诸多映射规则，Ingress Controller通过监听这些配置规则并转化为Nginx的反向代理配置，然后对外提供服务。   
>>  ○ Ingress：kubernetes中的一个对象，作用是定义请求如何转发到Service的规则。
>>  ○ Ingress Controller：具体实现反向代理及负载均衡的程序，对Ingress定义的规则进行解析，根据配置的规则来实现请求转发，实现的方式有很多，比如Nginx，Contour，Haproxy等。

>● Ingress（以Nginx）的工作原理如下：  
>>  ○ 用户编写Ingress规则，说明那个域名对应kubernetes集群中的那个Service。  
>>  ○ Ingress控制器动态感知Ingress服务规则的变化，然后生成一段对应的Nginx的反向代理配置。  
>>  ○ Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中，并动态更新。  
>>  ○ 到此为止，其实真正在工作的就是一个Nginx了，内部配置了用户定义的请求规则。  

### 5.4.1 准备  

- 搭建ingress  
```shell
mkdir ingress-controller
cd ingress-controller

# 下载ingress
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.30.0/deploy/static/mandatory.yaml

# 创建
kubectl apply -f ./

```

- 准备service和pod  
两个service，一个nginx，一个tomcat。分别惯着两个deployment，deoplyment下面各有3个pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
        - name: nginx
          image: ningx:1.17.1
          ports:
            - containerPort: 80

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
  namespace: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat-pod
  template:
    metadata:
      laebels:
        app: tomcat-pod
    spec:
      containers:
        - name: tomcat
          image: tomcat:8.6
          ports:
            - containerPort: 8080

---

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: dev
spec:
  selector:
    app: nginx-pod
  clusterIP: Nonde
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80

---

apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
  namespace: dev
spec:
  selector:
    app: tomcat-pod
  clusterIP: None
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
```

- Http代理  
```yaml
apiVersion: extensions/v1betal
kind: Ingress
metadata:
  name: ingress-http
  namespace: dev
spec:
  rules:
    - host: nginx.wahaha.com
      http:
        paths:
          - path: /
            backend:
              serviceName: nginx-service
              servicePort: 80
    - host: tomcat.wahaha.com
      http:
        paths:
          - path: /
            backend:
              serviceName: tomcat-service
              servicePort: 8080
```

# 6. 存储  
## 6.1 概述  
> 容器的生命周期可能很短，会被频繁的创建和销毁。那么容器在销毁的时候，保存在容器中的数据也会被清除。这种结果对用户来说，在某些情况下是不乐意看到的。为了持久化保存容器中的数据，kubernetes引入了Volume的概念。  

> Volume是Pod中能够被多个容器访问的共享目录，它被定义在Pod上，然后被一个Pod里面的多个容器挂载到具体的文件目录下，kubernetes通过Volume实现同一个Pod中不同容器之间的数据共享以及数据的持久化存储。Volume的生命周期不和Pod中的单个容器的生命周期有关，当容器终止或者重启的时候，Volume中的数据也不会丢失。

> kubernetes的Volume支持多种类型，比较常见的有下面的几个：  
>> 简单存储：EmptyDir、HostPath、NFS。  
>> 高级存储：PV、PVC。  
>> 配置存储：ConfigMap、Secret。

## 6.2 基本存储  
### 6.2.1 EmptyDir

就是一个host的空目录，随机分配，pod销毁时会被删除。  
> 主要作用：
>>   临时空间，例如用于某些应用程序运行时所需的临时目录，且无须永久保留。  
>>   一个容器需要从另一个容器中获取数据的目录（多容器共享目录）。

- 创建  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-emptydir
  namespace: dev
spec: 
  containers:
    - name: nginx
      image: nginx:1.17.1
      ports:
        - port: containerPort: 80
      volumeMounts:  # 将logs-volume挂载到nginx容器中对应的目录，该目录为/var/log/nginx
        - name: logs-volume
          mountPaht: /var/log
    - name: busybox
      image: busybos:1.30
      command: ["..."]
      volumeMount:
        - name: logs-volume
          mountPaht: /logs
  volumes: #  声明volume，name为logs-volume，类型为emptyDir
    - name: logs-volume
      emptyDir: {}
```

```shell
kubectl create -f volume-emptydir.yaml
```

- 查看  
```shell
kubectl get pod volume-emptydir -n dev -o wide
```

- 查看log  
```shell
# -f + pod的名字   -c + 容器的名字
kubectl logs -f volume-emptydir -n dev -c busybox
```

### 6.2.2 HostPath
- 概述  
EmptyDir中的数据不会被持久化，它会随着Pod的结束而销毁，如果想要简单的将数据持久化到主机中，可以选择HostPath。

- 创建  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-hostpath
  namespace: dev
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      imagePullPolicy: ifNotPresent
      ports:
        - containerPort: 80
      volumeMounts:
        - name: log-volume # 把log-volume（主机上的目录）挂在到容器里面的/var/log
          mountPath: /var/log
    - name: busybox
      image: busybox:1.30
      imagePullPolicy: Always
      command: ["..."]
      volumeMounts:
        - name: log-volume
          mountPaht: /logs
  volumes: # 声明volume，name是log-volume，挂在主机的目录/root/logs下
    - name: log-volume
      hostPath: # 类型为hostPath
        path: /root/logs
        type: DirectoryOrCreate # 目录存在就用，不存在就创建
```

> type的值的说明：  
● DirectoryOrCreate：目录存在就使用，不存在就先创建后使用。  
● Directory：目录必须存在。  
● FileOrCreate：文件存在就使用，不存在就先创建后使用。  
● File：文件必须存在。  
● Socket：unix套接字必须存在。  
● CharDevice：字符设备必须存在。  
● BlockDevice：块设备必须存在。  

```shell
kubectl create -f volume.hostpath.yaml
```

- 查看  
```shell
kubectl get pod volume-hostpath -n dev -o wide
```

### 6.2.3 NFS  
- 概述  

HostPath虽然可以解决数据持久化的问题，但是一旦Node节点故障了，Pod如果转移到别的Node节点上，又会出现问题，此时需要准备单独的网络存储系统，比较常用的是NFS和CIFS。

NFS是一个网络文件存储系统，可以搭建一台NFS服务器，然后将Pod中的存储直接连接到NFS系统上，这样，无论Pod在节点上怎么转移，只要Node和NFS的对接没有问题，数据就可以成功访问。

- 搭建nfs服务器  
```shell
# 在节点上安装nfs服务器
yum install -y nfs-utils rpcbind

# 准备一个共享目录
mkdir -pv /root/data/nfs

# 将共享目录以读写权限暴漏给k8s集群的所有主机
vim /etc/exports
/root/data/nfs ip地址(rw,no_root_squash)

# 修改权限 
chmod 777 -R /root/data/nfs

# 记载配置
exportffs -r

# 启动nfs服务
systemctl start rpcbind
systemctl enable rpcbind
systemctl start nfs
systemctl enable nfs

# 测试挂在是否成功
showmount -e ip地址

# 为驱动NFS设备，在k8s集群的节点上面安装nfs服务器
yum -y install nfs-utils

# 在集群街店上面测试挂在是否成功
showmount -e ip地址
```

- 创建pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-nfs
  namespace: dev
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 80
      volumeMounts:
        - name: log-volume
          mountPath: /var/log
    - name: busybox
      image: busybox:1.30
      imagePullPolicy: IfNotPresent
      command: ["..."]
      volumeMounts:
        - name: log-volume
          mountPath: /logs
  volumes: 
    - name: log-volume
      nfs:
        server: 0.0.0.0 # nfs服务器地址
        path: /root/data/nfs # 共享文件路径
```

```shell
kubectl create -f volume-nfs.yaml
```

- 查看  
```shell
kubectl get pod volume-nfs -n dev
```

## 6.3 高级存储  

### 6.3.1 PV/PVC概述    
> 使用NFS提供存储，此时就要求用户会搭建NFS系统，并且会在yaml配置nfs。由于kubernetes支持的存储系统有很多，要求客户全部掌握，显然不现实。为了能够屏蔽底层存储实现的细节，方便用户使用，kubernetes引入了PV和PVC两种资源对象。

> PV（Persistent Volume）是持久化卷的意思，是对底层的共享存储的一种抽象。一般情况下PV由kubernetes管理员进行创建和配置，它和底层具体的共享存储技术有关，并通过插件完成和共享存储的对接。

> PVC（Persistent Volume Claim）是持久化卷声明的意思，是用户对于存储需求的一种声明。换言之，PVC其实就是用户向kubernetes系统发出的一种资源需求申请。

> 使用了PV和PVC之后，工作可以得到进一步的提升：  
>> 存储：存储工程师维护。  
>> PV：kubernetes管理员维护。  
>> PVC：kubernetes用户维护。  

### 6.3.2 PV  

- 资源清单  
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2
spec:
  nfs: # 存储类型，和底层正则的存储对应
    path:
    server:
  capacity: # 存储能力，目前只支持存储空间的设置
    storage: 2Gi
  accessModes: # 访问模式
    - storageClassName: # 存储类别
      persistentVolumeReclaimPolicy: # 回收策略

```

> pv的关键配置参数说明：  
- 存储类型：底层实际存储的类型，kubernetes支持多种存储类型，每种存储类型的配置有所不同。  
- 存储能力（capacity）：目前只支持存储空间的设置（storage=1Gi），不过未来可能会加入IOPS、吞吐量等指标的配置。  
- 访问模式（accessModes）：  
  - 用来描述用户应用对存储资源的访问权限，访问权限包括下面几种方式：  
    - ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载。  
    - ReadOnlyMany（ROX）：只读权限，可以被多个节点挂载。  
    - ReadWriteMany（RWX）：读写权限，可以被多个节点挂载。  
  - 需要注意的是，底层不同的存储类型可能支持的访问模式不同。  
- 回收策略（ persistentVolumeReclaimPolicy）：  
  - 当PV不再被使用之后，对其的处理方式，目前支持三种策略：  
    - Retain（保留）：保留数据，需要管理员手动清理数据。  
    - Recycle（回收）：清除PV中的数据，效果相当于rm -rf /volume/*。  
    - Delete（删除）：和PV相连的后端存储完成volume的删除操作，常见于云服务器厂商的存储服务。  
  - 需要注意的是，底层不同的存储类型可能支持的回收策略不同。  
- 存储类别（storageClassName）：PV可以通过storageClassName参数指定一个存储类别。  
  - 具有特定类型的PV只能和请求了该类别的PVC进行绑定。  
  - 未设定类别的PV只能和不请求任何类别的PVC进行绑定。  
- 状态（status）：一个PV的生命周期，可能会处于4种不同的阶段。  
  - Available（可用）：表示可用状态，还未被任何PVC绑定。  
  - Bound（已绑定）：表示PV已经被PVC绑定。  
  - Released（已释放）：表示PVC被删除，但是资源还没有被集群重新释放。  
  - Failed（失败）：表示该PV的自动回收失败。  


- 准备工作（NFS环境）  
```shell
# 创建目录
mkdir -pv /root/data/{pv1, pv2, pv3}

# 授权  
chmod 777 -R /root/data

# 修改/etc/exports
vim /etc/exports
/root/data/pv1     IP address(rw,no_root_squash) 
/root/data/pv2     IP address(rw,no_root_squash) 
/root/data/pv3     IP address(rw,no_root_squash)

# 重启nfs服务
systemctl restart nfs
```

- 创建PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  nds: 
    path: /root/data/pv1
    server: 0.0.0.0
  capacity: # 存储能力，目前只支持存储空间的设置
    storage: 1Gi
  accessModes: 
    - ReadWritMany
  persistentVolumeReclaimPolicy: Retain

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  nds: 
    path: /root/data/pv2
    server: 0.0.0.0
  capacity: # 存储能力，目前只支持存储空间的设置
    storage: 2Gi
  accessModes: 
    - ReadWritMany
  persistentVolumeReclaimPolicy: Retain

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  nds: 
    path: /root/data/pv3
    server: 0.0.0.0
  capacity: # 存储能力，目前只支持存储空间的设置
    storage: 3Gi
  accessModes: 
    - ReadWritMany
  persistentVolumeReclaimPolicy: Retain
```

```shell
kubectl create -f pv.yaml
```

- 查看pv
```shell
kubectl get pv -o wide
```

### 6.3.3 PVC
PVC是资源的申请，用来声明对存储空间、访问模式、存储类别需求信息

- 资源清单  
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
  namespace: dev
spec:
  accessModes: # 访问模式
    - ReadWriteMany
  selector: 采用标签对pv进行选择
  storageClassNme: # 存储类别
  resources: # 请求类别
    requests:
      storage: 5Gi
```

> PVC的关键配置参数说明：  
- 访客模式（accessModes）：用于描述用户应用对存储资源的访问权限。  
- 用于描述用户应用对存储资源的访问权限：  
  - 选择条件（selector）：通过Label Selector的设置，可使PVC对于系统中已存在的PV进行筛选。
  - 存储类别（storageClassName）：PVC在定义时可以设定需要的后端存储的类别，只有设置了该class的pv才能被系统选出。
  - 资源请求（resources）：描述对存储资源的请求。


- 创建pvc
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  namespace: dev
spec:
  accessModes: 
    - ReadWriteMany
  resources:
    request:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  namespace: dev
spec:
  accessModes: 
    - ReadWriteMany
  resources:
    request:
      storage: 2Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
  namespace: dev
spec:
  accessModes: 
    - ReadWriteMany
  resources:
    request:
      storage: 3Gi
```

```shell
kubectl create -f pvc.yaml
```

- 查看pvc
```
kubectl get pvc -n dev -o wide
```

- 使用pod创建pvc
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: dev
spec:
  containers:
    - name: busybox
      image: busybox:1.30
      command: ["..."]
      volumeMounts:
        - name: volume
          mountPath: /root
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc1
        readOnly: false
---
apiVersion:
kind: Pod
metadata:
  name: pod2
  namespace: dev
spec: 
  containers:
    - name: busybox
      image: busybox:1.30
      command: [".."]
      volumeMounts:
        - name: volume
          mountPath: /root/
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc2
        readOnly: false
```

## 6.4 配置存储
### 6.4.1 ConfigMap  
- 概述  
主要作用是用来存储配置信息  

- 资源清单  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap
  namespace:
data: # <map[string]string>
...
```

- 创建  
```yaml
apiVersion: v1
kind: ConfigMap
metadata: 
  name: configmap
  namespace: dev
data:
  info:
    username: root
    password: root
```
```shell
kubectl create -f configmap.yaml
```

- 创建pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-configmap
  namespace: dev
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      volumeMounts:
        - name: config
          mountPath: /configmap/config
  volumes:
    - name: config
      configMap:
        name: configmap
```

```shell
kubectl create -f pod-configmap.yaml
```

- 查看pod  
```shell
kubectl get pod pod-configmap.yaml
```

### 6.4.2 Secret
- 概述  
用来存储敏感信息，比如密码，密钥，证书等  

- 创建secrete  
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret
  namespace: dev
type: Opaque
data:
  username: admin
  password: admin
```

```shell
kubectl create -f secret.yaml
```

- 查看secret详情  
```shell
kubectl describe secret secret -n dev
```

- 创建pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  namespace: dev
spec:
  containers:
    - name: nginx
      image: nginx:1.17.1
      volumeMounts:
        - name: config
          mountPath: /secret/config
  volumes:
    - name: config
      secret:
        secretName: secret
```

```shell
kubectl crete -f pod-secret.yaml
```

### 6.4.3 ConfigMap高级

- 概述  

> 注意事项：  
>> ConfigMap 在设计上不是用来保存大量数据的。在 ConfigMap 中保存的数据不可超过 1 MiB。  
>> 如果需要保存超出此尺寸限制的数据，需要考虑挂载存储卷或者使用独立的数据库或者文件服务。  

语法
```shell
kubectl create configmap <map-name> <data-source>
```

