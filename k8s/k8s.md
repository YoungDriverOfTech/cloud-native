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
apiVersion: v1
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

