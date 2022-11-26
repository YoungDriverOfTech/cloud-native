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

### 2.2.1 查  
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

### 2.2.2 增  
```shell
liangxiaole@ryoushous-MBP yaml % kubectl create ns dev
namespace/dev created
liangxiaole@ryoushous-MBP yaml % 
```

### 2.2.3 删除  
```shell
liangxiaole@ryoushous-MBP yaml % kubectl delete ns dev
namespace "dev" deleted
```
\
;