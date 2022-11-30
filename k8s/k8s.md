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
