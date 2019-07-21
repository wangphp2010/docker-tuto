

git init
git add README.md
git commit -m "first commit"

git remote add origin https://github.com/wangphp2010/docker-tuto.git

git push -u origin master

https://hijiangtao.github.io/2018/04/17/Docker-in-Action/

## 安装

```bash

# 安装
$ wget -qO- https://get.docker.com/ | sh

# 启动docker 后台服务
$ sudo service docker start / sudo systemctl start docker

#建立 docker 组：
$ sudo groupadd docker

#将当前用户加入 docker 组：
$ sudo usermod -aG docker $USER

# 退出当前终端并重新登录，进行如下测试 测试运行hello-world
$ docker run hello-world

```



```bash
# 拉取 镜像 这些镜像是官方的
# Docker 镜像仓库地址若不写则默认为 Docker Hub，而仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像
$ docker pull ubuntu 

# 查看镜像
$ docker image ls
```


 
```bash
# 运行 镜像 
$ docker run -it --rm   ubuntu:16.04  bash
```
+ -it：这是两个参数，一个是 -i 表示交互式操作，一个是 -t 为终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
+ --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。
+ ubuntu:16.04：这是指用 ubuntu:16.04 镜像为基础来启动容器。
+ bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用 bash。

## Docker 镜像还有一些常用操作，比如：

+ docker image ls - 列出本地已下载镜像
+ docker image rm [选项] <镜像1> [<镜像2> ...] - 删除镜像
+ docker logs <id/container_name> - 查看容器日志
+ docker ps - 列出当前所有正在运行的容器
+ docker ps -l - 列出最近一次启动的容器
+ docker search image_name - 从 Docker Hub 检索镜像
+ docker history image_name - 显示镜像历史
+ docker push new_image_name - 发布镜像



```bash
# 删除镜像 
$ docker image rm -f [image ID] 



```

## 通过 commit 命令理解镜像构成

```bash
 
# nginx 镜像启动一个容器，命名为 webserver，并映射在 4000 端口 浏览器去访问它
# 如果是虚拟机 将 localhost 换为虚拟机地址 http://192.168.0.40:4000/
$ docker run --name webserver -d -p 4000:80 nginx

# docker exec 命令进入容器，修改其内容
$ docker exec -it webserver bash
$ root@7603bd94b5e:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
$ root@27603bd94b5e:/# exit

# docker diff 命令看到具体的改动
$ docker diff webserver
```

## 假设以上操作已经符合我们的定制了，接下来就将它保存下来。commit 语法格式如下
### docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

```bash
$ docker commit \
    --author "Joe Jiang <hijiangtao@gmail.com>" \
    --message "modify: Nginx default page to Hello Docker" \
    webserver \
    nginx:v2
    
$ docker image ls

# 输出
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
nginx               v2                  48bcb7b903a4        About a minute ago   109MB
ubuntu              16.04               c9d990395902        4 days ago           113MB
hello-world         latest              e38bc07ac18e        5 days ago           1.85kB
nginx               latest              b175e7467d66        6 days ago           109MB

```
## 查看这个镜像最新版的历史记录

```bash

$ docker history nginx:v2

# 输出
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
48bcb7b903a4        3 minutes ago       nginx -g daemon off;                            135B                modify: Nginx default page to Hello Docker
b175e7467d66        6 days ago          /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>           6 days ago          /bin/sh -c #(nop)  STOPSIGNAL [SIGTERM]         0B
...
``` 

```bash
# 查看所有容器
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                  NAMES
27603bd94b5e        nginx               "nginx -g 'daemon of…"   18 minutes ago      Up 18 minutes            0.0.0.0:4000->80/tcp   webserver
fdb4b0c9a5e1        hello-world         "/hello"                 2 hours ago         Exited (0) 2 hours ago                          angry_bhabha

# 终止 webserver
$ docker container stop webserver
webserver

# 再次查看所有容器，确保 webserver 终止
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
27603bd94b5e        nginx               "nginx -g 'daemon of…"   21 minutes ago      Exited (0) 27 seconds ago                       webserver
fdb4b0c9a5e1        hello-world         "/hello"                 2 hours ago         Exited (0) 2 hours ago                          angry_bhabha
 
# 运行commit的镜像 -d 后台运行
$ docker run --name web2 -d -p 4001:80 nginx:v2

```

## -d 参数的解释

```bash
$ docker run ubuntu:16.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"

# 输出
hello world
hello world
hello world
hello world
```

> -d Run container in background and print container ID

```bash
# 如果我们在如上命令中加入 -d 命令，那么这些 hello world 信息就需要通过如下命令来查看了，其中 ID 是容器的名称：
$ docker logs ID

```


## 利用 Dockerfile 定制镜像



```bash
# 在本机上登陆你的 Docker 账户，以便之后发布使用，如果没有请到 https://cloud.docker.com/ 注册一个。本机登陆命令：
$ docker login 


$ mkdir imageDocker
$ cd imageDocker 
$ nano dockerfile


# 然后在当前目录新建一个 Dockerfile 文件，内容如下所示：
# Dockerfile

# 使用 Python 运行时作为基础镜像
FROM python:2.7-slim

# 设置 /app 为工作路径
WORKDIR /app

# 将当前目录所有内容复制到容器的 /app 目录下
ADD . /app

# 安装 requirements.txt 中指定的包
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 对容器外开放80端口
EXPOSE 80

# 定义环境变量
ENV NAME World

# 当容器启动时运行 app.py 
CMD ["python", "app.py"]
```

> 以上内容中，我们用 FROM 指定基础镜像，在 Docker Store 上有非常多的高质量的官方镜像，服务类镜像如 nginx、redis、mongo、mysql、httpd、php、tomcat 等；各种语言应用镜像如 node、openjdk、python、ruby、golang 等；操作系统镜像如 ubuntu、debian、centos、fedora、alpine。除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 scratch。直接 FROM scratch 会让镜像体积更加小巧。

> RUN 的正确写法: 否则每行RUN 都会增加一层镜像 导致系统浪费甚至崩溃

```bash
RUN buildDeps='gcc libc6-dev make' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

requirements.txt 内容如下：

```bash
Flask
Redis
```
app.py 内容如下：

```bash
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

### 构建这个镜像

```bash
# docker build [选项] <上下文路径/URL/->
# docker build 得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎

$ docker build -t friendlyhello .

# 运行刚才的镜像
$ docker run -p 4000:80 friendlyhello
# 浏览器去访问它 http://192.168.0.40:4000/  

# 也可以在终端查看
$ http://192.168.0.40:4000/
```

## 发布这个镜像到 Docker Hub

```bash
# tag 格式
docker tag image username/repository:tag

# 本例中的运行命令
docker tag friendlyhello wang2010/friendlyhello:v1
```

> 接着将它发布：


```bash
# push 格式
docker push username/repository:tag

# 本例中的运行命令
docker push wang2010/friendlyhello:v1
```

### 至此，大功告成。现在你可以在任意一台配有 docker 环境的机器上运行该镜像了。
```
$ docker run -p 4000:80 wang2010/friendlyhello:v1

```


###  参考

+ [Docker](https://en.wikipedia.org/wiki/Docker_(software) "Docker")
+ [Docker 从入门到实践](https://legacy.gitbook.com/book/yeasy/docker_practice "Docker 从入门到实践")
+ [Docker容器与虚拟机区别]( http://www.cnblogs.com/pangguoping/articles/5515286.html "Docker容器与虚拟机区别")
+ [Docker三剑客实践之部署集群](  http://marklux.cn/blog/55 "Docker三剑客实践之部署集群")
+ [Get Started with Docker]( https://docs.docker.com/get-started/part2/ "Get Started with Docker")








