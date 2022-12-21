# Dockerfile  
```shell
# dockerfile

FROM node:16-alpine3.15

WORKDIR /home/app/

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8080

ENTRYPOINT ["npm", "run"]

CMD ["serve"]
```

## 1. From 
这次构建的最基本的镜像，同时必须是第一条指令，因为后续的所有操作必须都基于这个基础景象之上，每一次运行系统的命令都必须是基础镜像中包含的，所以当基础镜像中没有`Node`环境的话，后续命令也无法正常使用`Node的API`。

## 2. WORKDIR 
设置当前的工作目录，没有就创建。所有操作的路径都是在这个路径下完成的。

## 3. COPY  
复制，但是只能复制宿主机本体的文件。

## 4. ADD
与COPY类似，但是更加强大，可以复制宿主机以外的能容。类似于linux的scp。

## 5. EXPOSE
声明当前`容器`要访问的网络端口，但是仅仅是声明需要暴露的端口，启动的时候并不会直接能访问到，需要在启动容器的时候添加`-p`参数，挂载宿主机端口。

## 6. RUN
在构建镜像的过程中执行的命令，需要执行的命令格式如下。
```shell
# 直接执行shell命令
RUN <command>

# 启动文件（或者全局命令模式，例如 npm） + 传入参数的格式，如果 shell 命令不太容易满足功能的时候，可以使用第二种，直接写好处理的脚本，加传入参数即可。
RUN ['executable', 'param']
```

## 7. CMD
```shell
CMD <command>
CMD ['executable', 'param']
```
在容器启动时候执行的命令。

### RUN & CMD
两个都是执行命令，`RUN`是构建镜像时就运行的命令，CMD是容器启动时执行的命令，在构建时候并不运行。

此外 Docker 容器默认会把容器内部第一个进程，也就是 pid=1 的程序作为 Docker 容器是否正在运行的依据，所以为了 Docker 容器常驻，CMD 执行的 Node 服务（其他服务也是一样）的时候一定要前台启动，否则使用后台启动的话，容器启动后执行完命令就会自动退出。

## 8. ENTRYPOINT
`ENTRYPOINT`是指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有其他传入值作为该命令的参数，单独使用时与CMD脚本使用方式一致。
```shell
CMD ['executable', 'param1', 'param1']
```

当与 `CMD` 命令混合使用时，`CMD` 的功能就不再是直接运行脚本，而是将 `CMD 的内容`作为参数传给 `ENTRYPOINT`，所以在上述示例的 dockerfole 中大家会发现两种混合的写法：ENTRYPOINT 的参数是 `NPM RUN`，而 CMD 的参数是 `serve`。

## 9. Example
真实项目中当然不会有 dev-server 这种来访问静态资源，所以我们需要一个像 Nginx 的静态服务器来帮助访问静态资源，但是我们既然使用了 Node 环境如果要使用 Nginx 的话，就需要切换基础镜像或者安装一个 Nginx，所以本示例采用 anywhere 库来作为静态服务。

> anywhere 本身就是前台启动服务，如果想使用 Nginx 来作为静态服务器的话，记得添加 daemon off 参数来强制 Nginx 进程前台启动。

```shell
FROM node:16-alpine3.15
RUN mkdir -p /home/adpp/
WORKDIR /home/app/
RUN npm i anywhere -g
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 8080
CMD anywhere -p 8080 -d ./dist -s
```

```shell
# 创建镜像
docker build -f ./Dockerfile -f test:0.0.1

# 启动容器
docker run -d -p 8080 test:0.0.1
```