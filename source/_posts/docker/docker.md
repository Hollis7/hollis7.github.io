---
title: docker初学
categories:
  - docker
tags:
  - docker
mathjax: true
---

<meta name="referrer" content="no-referrer"/>

docker的一些基本知识

<!--more-->

## 参考链接

[Docker — 从入门到实践](https://docker-practice.github.io/zh-cn/)

## docker初识

**Docker** 使用 `Google` 公司推出的 [Go 语言](https://golang.google.cn/) 进行开发实现，基于 `Linux` 内核的 [cgroup](https://zh.wikipedia.org/wiki/Cgroups)，[namespace](https://en.wikipedia.org/wiki/Linux_namespaces)，以及 [OverlayFS](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) 类的 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 等技术，对进程进行封装隔离，属于 [操作系统层面的虚拟化技术](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。最初实现是基于 [LXC](https://linuxcontainers.org/lxc/introduction/)，从 `0.7` 版本以后开始去除 `LXC`，转而使用自行开发的 [libcontainer](https://github.com/docker/libcontainer)，从 `1.11` 版本开始，则进一步演进为使用 [runC](https://github.com/opencontainers/runc) 和 [containerd](https://github.com/containerd/containerd)。

传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

## 基本概念

### Docker 镜像

操作系统分为 **内核** 和 **用户空间**。对于 `Linux` 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支持。而 **Docker 镜像**（`Image`），就相当于是一个 `root` 文件系统。比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 Ubuntu 18.04 最小系统的 `root` 文件系统。

**Docker 镜像** 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 **不包含** 任何动态数据，其内容在构建之后也不会被改变。

### 镜像

镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

## 建立 docker 用户组

建立 `docker` 组：

```bash
sudo groupadd docker
```

使用以下命令来查看 Docker 用户组的成员：

~~~bash
grep docker /etc/group
~~~

将当前用户加入 `docker` 组：

```bash
sudo usermod -aG docker $USER
```

退出当前终端并重新登录，进行如下测试。

## 使用 Docker 镜像

### 获取

[Docker Hub](https://hub.docker.com/search?q=&type=image) 上有大量的高质量的镜像可以用

从 Docker 镜像仓库获取镜像的命令是 `docker pull`。其命令格式为：

~~~bash
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
~~~

如：

~~~bash
docker pull ubuntu:18.04
~~~

### 运行

```bash
docker run -it --rm ubuntu:18.04 bash
```

- `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。
- `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间。
- `ubuntu:18.04`：这是指用 `ubuntu:18.04` 镜像为基础来启动容器。
- `bash`：放在镜像名后的是 **命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。

## 列出镜像

~~~bash
docker image ls
~~~

### 镜像体积

可以通过 `docker system df` 命令来便捷的查看镜像、容器、数据卷所占用的空间。

~~~bash
docker system df
~~~

### 虚悬镜像

~~~bash
<none>               <none>              00285df0df87        5 days ago          342 MB
~~~

一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除。

```bash
docker image prune
```

### 中间层镜像

为了加速镜像构建、重复利用资源，Docker 会利用 **中间层镜像**。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。默认的 `docker image ls` 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加 `-a` 参数。

```bash
docker image ls -a
```

## 删除本地镜像

如果要删除本地的镜像，可以使用 `docker image rm` 命令，其格式为：

```bash
$ docker image rm [选项] <镜像1> [<镜像2> ...]
```

### 短id

我们可以用镜像的完整 ID，也称为 `长 ID`，来删除镜像。使用脚本的时候可能会用长 ID，但是人工输入就太累了，所以更多的时候是用 `短 ID` 来删除镜像。`docker image ls` 默认列出的就已经是短 ID 了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。

~~~bash
docker image rm 501
~~~

### 镜像名

~~~bash
docker image rm centos
~~~

## 使用 Dockerfile 定制镜像

在一个空白目录中，建立一个文本文件，并命名为 `Dockerfile`：

~~~bash
mkdir mynginx
cd mynginx
touch Dockerfile
~~~

其内容为：

~~~dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
~~~

> 这个 Dockerfile 指定了使用 nginx 镜像作为基础镜像，并在容器启动时执行了一个命令来创建一个包含 "Hello, Docker!" 的 HTML 文件。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

```docker
FROM scratch
```

### 构建镜像

在 `Dockerfile` 文件所在目录执行：

```bash
docker build -t nginx:v3 .
```

从标准输入中读取上下文压缩包进行构建

```bash
docker build - < context.tar.gz
```

## Dockerifle参考

### seal-python

~~~dockerfile
FROM ubuntu:22.04

# define the folder where our src should exist/ be deposited
ARG SRC=/python-seal

# prevents update and install asking for tz
ENV DEBIAN_FRONTEND=noninteractive

# install dependencies
RUN apt update && \
    apt install -y git build-essential cmake python3 python3-dev python3-pip && \
    mkdir -p ${SRC}

# copy into container requirements and install them before rest of code
RUN pip3 install numpy pybind11

# copy everything into container now that requirements stage is complete
COPY . ${SRC}

# setting our default directory to the one specified above
WORKDIR ${SRC}

# update submodules
RUN cd ${SRC} && \
    git submodule update --init --recursive
    # git submodule update --remote

# build and install seal + bindings
RUN cd ${SRC}/SEAL && \
    cmake -S . -B build -DSEAL_USE_MSGSL=OFF -DSEAL_USE_ZLIB=OFF -DSEAL_USE_ZSTD=OFF && \
    cmake --build build && \
    cd ${SRC} && \
    python3 setup.py build_ext -i

CMD ["/usr/bin/python3"]
~~~

## Dockerfile 指令详解

### COPY 复制文件

格式：

- `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。比如：

```docker
COPY package.json /usr/src/app/
```

`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR` 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

~~~bash
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
~~~

如果源路径为文件夹，复制的时候不是直接复制该文件夹，而是将文件夹中的内容复制到目标路径。

### CMD 容器启动命令

`CMD` 指令的格式和 `RUN` 相似，也是两种格式：

- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
- 参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如，`ubuntu` 镜像默认的 `CMD` 是 `/bin/bash`，如果我们直接 `docker run -it ubuntu` 的话，会直接进入 `bash`。我们也可以在运行时指定运行别的命令，如 `docker run -it ubuntu cat /etc/os-release`。这就是用 `cat /etc/os-release` 命令替换了默认的 `/bin/bash` 命令了，输出了系统版本信息。



如果使用 `shell` 格式的话，实际的命令会被包装为 `sh -c` 的参数的形式进行执行。比如：

```docker
CMD echo $HOME
```

在实际执行中，会将其变更为：

```docker
CMD [ "sh", "-c", "echo $HOME" ]
```

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 `systemd` 去启动后台服务，容器内没有后台服务的概念。

一些初学者将 `CMD` 写为：

```docker
CMD service nginx start
```

然后发现容器执行后就立即退出了。甚至在容器内去使用 `systemctl` 命令结果却发现根本执行不了。这就是因为没有搞明白前台、后台的概念，没有区分容器和虚拟机的差异，依旧在以传统虚拟机的角度去理解容器。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

而使用 `service nginx start` 命令，则是希望 upstart 来以后台守护进程形式启动 `nginx` 服务。而刚才说了 `CMD service nginx start` 会被理解为 `CMD [ "sh", "-c", "service nginx start"]`，因此主进程实际上是 `sh`。那么当 `service nginx start` 命令结束后，`sh` 也就结束了，`sh` 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 `nginx` 可执行文件，并且要求以前台形式运行。比如：

```docker
CMD ["nginx", "-g", "daemon off;"]
```

### ENTRYPOINT 入口点

#### 场景一：让镜像变成像命令一样使用

假设我们需要一个得知自己当前公网 IP 的镜像，那么可以先用 `CMD` 来实现：

```docker
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
CMD [ "curl", "-s", "http://myip.ipip.net" ]
```

假如我们使用 `docker build -t myip .` 来构建镜像的话，如果我们需要查询当前公网 IP，只需要执行：

```bash
$ docker run myip
```

这么看起来好像可以直接把镜像当做命令使用了，不过命令总有参数，如果我们希望加参数呢？比如从上面的 `CMD` 中可以看到实质的命令是 `curl`，那么如果我们希望显示 HTTP 头信息，就需要加上 `-i` 参数。那么我们可以直接加 `-i` 参数给 `docker run myip` 么？

```bash
$ docker run myip -i
docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable file not found in $PATH\"\n".
```

我们可以看到可执行文件找不到的报错，`executable file not found`。之前我们说过，跟在镜像名后面的是 `command`，运行时会替换 `CMD` 的默认值。因此这里的 `-i` 替换了原来的 `CMD`，而不是添加在原来的 `curl -s http://myip.ipip.net` 后面。而 `-i` 根本不是命令，所以自然找不到。

那么如果我们希望加入 `-i` 这参数，我们就必须重新完整的输入这个命令：

```bash
$ docker run myip curl -s http://myip.ipip.net -i
```

这显然不是很好的解决方案，而使用 `ENTRYPOINT` 就可以解决这个问题。现在我们重新用 `ENTRYPOINT` 来实现这个镜像：

```docker
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
```

这次我们再来尝试直接使用 `docker run myip -i`：



### ENV 设置环境变量

格式有两种：

- `ENV <key> <value>`
- `ENV <key1>=<value1> <key2>=<value2>...`

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如 `RUN`，还是运行时的应用，都可以直接使用这里定义的环境变量。

```docker
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。比如在官方 `node` 镜像 `Dockerfile` 中，就有类似这样的代码：

```docker
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-x64.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs
```

在这里先定义了环境变量 `NODE_VERSION`，其后的 `RUN` 这层里，多次使用 `$NODE_VERSION` 来进行操作定制。可以看到，将来升级镜像构建版本的时候，只需要更新 `7.2.0` 即可，`Dockerfile` 构建维护变得更轻松了。

### ARG 构建参数

格式：`ARG <参数名>[=<默认值>]`

构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，`ARG` 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的。

`Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖。

> 这样做的目的通常是在构建镜像时提供一些动态的值，例如镜像版本、构建环境等，而不将这些敏感信息暴露给容器的运行时环境。相比之下，使用 `ENV` 指令设置的环境变量会在容器运行时持续存在。

ARG 指令有生效范围，如果在 `FROM` 指令之前指定，那么只能用于 `FROM` 指令中。

```dockerfile
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library

FROM ${DOCKER_USERNAME}/alpine

# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library

RUN set -x ; echo ${DOCKER_USERNAME}
```

构建时会输出：`=> CACHED [2/2] RUN set -x ; echo library`

`docker run ...` 不会有输出结果

### VOLUME 定义匿名卷

为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 `Dockerfile` 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```docker
VOLUME /app/data
```

`data` 目录就会在容器运行时自动挂载为匿名卷，任何向 `/data` 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。当然，运行容器时可以覆盖这个挂载设置。比如：

```bash
docker run -it --rm -v $(pwd)/data:/app/data work_dir
```

这里的宿主机和容器一定要是绝对路径

~~~dockerfile
FROM ubuntu:22.04
WORKDIR /app
RUN mkdir -p /app/data  # 创建 data 目录
VOLUME /app/data  # 声明挂载点
RUN echo "hello" > world.txt
RUN touch a.bxt
~~~

### EXPOSE 声明端口

格式为 `EXPOSE <端口1> [<端口2>...]`。

`EXPOSE` 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。

要将 `EXPOSE` 和在运行时使用 `-p <宿主端口>:<容器端口>` 区分开来。`-p`，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

### WORKDIR 指定工作目录

```docker
RUN cd /app
RUN echo "hello" > world.txt
```

如果将这个 `Dockerfile` 进行构建镜像运行后，会发现找不到 `/app/world.txt` 文件，或者其内容不是 `hello`。原因其实很简单，在 Shell 中，连续两行是同一个进程执行环境，因此前一个命令修改的内存状态，会直接影响后一个命令；**而在 `Dockerfile` 中，这两行 `RUN` 命令的执行环境根本不同，是两个完全不同的容器。这就是对 `Dockerfile` 构建分层存储的概念不了解所导致的错误。**

之前说过每一个 `RUN` 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 `RUN cd /app` 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。

因此如果需要改变以后各层的工作目录的位置，那么应该使用 `WORKDIR` 指令。

~~~dockerfile
FROM ubuntu:22.04
WORKDIR /app

RUN echo "hello" > world.txt
~~~

构建和运行

~~~bash
docker build -t workdir_demo .
docker run -it --rm workdir_demo
~~~

建议多使用`--rm`，避免容器越来越多

### USER 指定当前用户

USER 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

~~~dockerfile
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
~~~

如果以 `root` 执行的脚本，在执行期间希望改变身份，比如希望以某个已经建立好的用户来运行某个服务进程，不要使用 `su` 或者 `sudo`，这些都需要比较麻烦的配置，而且在 TTY 缺失的环境下经常出错。建议使用 [`gosu`](https://github.com/tianon/gosu)。

~~~dockerfile
# 建立 redis 用户，并使用 gosu 换另一个用户执行命令
RUN groupadd -r redis && useradd -r -g redis redis
# 下载 gosu
RUN wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/1.12/gosu-amd64" \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true
# 设置 CMD，并以另外的用户执行
CMD [ "exec", "gosu", "redis", "redis-server" ]
~~~

### HEALTHCHECK 健康检查

- `HEALTHCHECK [选项] CMD <命令>`：设置检查容器健康状况的命令
- `HEALTHCHECK NONE`：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常，这是 Docker 1.12 引入的新指令。

和 `CMD`, `ENTRYPOINT` 一样，`HEALTHCHECK` 只可以出现一次，如果写了多个，只有最后一个生效。

~~~dockerfile
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
~~~

这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，并且使用 `curl -fs http://localhost/ || exit 1` 作为健康检查命令。

构建这个镜像

~~~bash
 docker build -t myweb:v1 .
~~~

构建好了后，我们启动一个容器

~~~bash
docker run -d --name web -p 80:80 --rm myweb:v1
~~~

在等待几秒钟后，再次 `docker container ls`，就会看到健康状态变化为了 `(healthy)`：

为了帮助排障，健康检查命令的输出（包括 `stdout` 以及 `stderr`）都会被存储于健康状态里，可以用 `docker inspect` 来查看。

~~~bash
docker inspect --format '{{json .State.Health}}' web | python -m json.tool
~~~

停止容器：

~~~bash
docker stop web
~~~

### ONBUILD 为他人做嫁衣裳

`ONBUILD` 是一个特殊的指令，它后面跟的是其它指令，比如 `RUN`, `COPY` 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

**基础镜像**

~~~dockerfile
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
~~~

**应用镜像**

~~~dockerfile
FROM my-node
~~~

当在各个项目目录中，用这个只有一行的 `Dockerfile` 构建镜像时，之前基础镜像的那三行 `ONBUILD` 就会开始执行，成功的将当前项目的代码复制进镜像、并且针对本项目执行 `npm install`，生成应用镜像。

### LABEL 指令

`LABEL` 指令用来给镜像以键值对的形式添加一些元数据（metadata）。

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

我们还可以用一些标签来申明镜像的作者、文档地址等：

```dockerfile
LABEL org.opencontainers.image.authors="yeasy"
LABEL org.opencontainers.image.documentation="https://yeasy.gitbooks.io"
```

### SHELL 指令

格式：`SHELL ["executable", "parameters"]`

SHELL` 指令可以指定 `RUN` `ENTRYPOINT` `CMD` 指令的 shell，Linux 中默认为 `["/bin/sh", "-c"]

## 容器

容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。

### 启动

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`exited`）的容器重新启动。

因为 Docker 的容器实在太轻量级了，很多时候用户都是随时删除和新创建容器。

#### 新建并启动

所需要的命令主要为 `docker run`。

例如，下面的命令输出一个 “Hello World”，之后终止容器。

```bash
$ docker run ubuntu:18.04 /bin/echo 'Hello world'
Hello world
```

#### 启动已终止容器

可以利用 `docker container start` 命令，直接将一个已经终止（`exited`）的容器启动运行。

### 后台运行

更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

```bash
 docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

要获取容器的输出信息，可以通过 `docker container logs` 命令。

```bash
docker container logs [container ID or NAMES]
```

~~~bash
docker container logs web
~~~

### 终止

可以使用 `docker container stop` 来终止一个运行中的容器。

~~~bash
docker container stop web
~~~

终止状态的容器可以用 `docker container ls -a` 命令看到。

~~~bash
docker container ls -a
~~~

### 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。

某些时候需要进入容器进行操作，包括使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令，原因会在下面说明。

~~~bash
docker run -dit ubuntu:18.04
-------------------------------------------
(base) ➜  container_demo docker container ls
CONTAINER ID   IMAGE                                     COMMAND                   CREATED         STATUS                PORTS                                   NAMES
326c1f3e8643   ubuntu:18.04                              "/bin/bash"               6 seconds ago   Up 5 seconds                                                  funny_lovelace
-------------------------------------------
docker exec -it 326 bash
~~~

如果从这个 stdin 中 exit，不会导致容器的停止（docker attach会停止容器）。这就是为什么推荐大家使用 `docker exec` 的原因。

### 导入和导出

#### 导出容器

如果要导出本地某个容器，可以使用 `docker export` 命令。

~~~bash
docker export bb5 > ubuntu18.tar
~~~

#### 导入容器快照

可以使用 `docker import` 从容器快照文件中再导入为镜像，例如

```bash
cat ubuntu18.tar | docker import - test/ubunntu:1.0
```

启动镜像

~~~bash
docker run -dit --name test_ubuntu --rm test/ubuntu:1.0 bash
911ae74f13732a0ff1b76cb44e174313dd6ccbe96a4a168b1b288b5a03ffd38c
~~~

进入镜像

~~~bash
docker exec -it 911 bash
docker exec -it test_ubuntu bash
~~~

停止容器

~~~bash
docker container stop test_ubuntu 
~~~

### 删除

可以使用 `docker container rm` 来删除一个处于终止状态的容器。例如

```bash
$ docker container rm trusting_newton
```

清理所有处于终止状态的容器，用 `docker container ls -a` 命令可以查看所有已经创建的包括终止状态的容器，如果数量太多要一个个删除可能会很麻烦，用下面的命令可以清理掉所有处于终止状态的容器。

~~~bash
docker container prune
~~~

## 访问仓库

### docker hub

#### 登录

可以通过执行 `docker login` 命令交互式的输入用户名及密码来完成在命令行界面登录 Docker Hub。

你可以通过 `docker logout` 退出登录。

#### 查找

可以通过 `docker search` 命令来查找官方仓库中的镜像，并利用 `docker pull` 命令来将它下载到本地。

另外，在查找的时候通过 `--filter=stars=N` 参数可以指定仅显示收藏数量为 `N` 以上的镜像。

#### 拉取镜像

~~~bash
docker pull centos
~~~

#### 推送镜像

用户也可以在登录后通过 `docker push` 命令来将自己的镜像推送到 Docker Hub。

以下命令中的 `username` 请替换为你的 Docker 账号用户名。

```bash
docker tag ubuntu:18.04 username/ubuntu:18.04
```

### 私有仓库

有时候使用 Docker Hub 这样的公共仓库可能不方便，用户可以创建一个本地仓库供私人使用。

#### 安装运行 docker-registry

## Docker 数据管理

 Docker 内部以及容器之间管理数据，在容器中管理数据主要有两种方式：

- 数据卷（Volumes）
- 挂载主机目录 (Bind mounts)

### 数据卷

`数据卷` 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

- `数据卷` 可以在容器之间共享和重用
- 对 `数据卷` 的修改会立马生效
- 对 `数据卷` 的更新，不会影响镜像
- `数据卷` 默认会一直存在，即使容器被删除

#### 创建一个数据卷

```bash
docker volume create my-vol
```

查看所有的 `数据卷`

```bash
docker volume ls

DRIVER              VOLUME NAME
local               my-vol
```

在主机里使用以下命令可以查看指定 `数据卷` 的信息

```bash
$ docker volume inspect my-vol
```

#### 启动一个挂载数据卷的容器

在用 `docker run` 命令的时候，使用 `--mount` 标记来将 `数据卷` 挂载到容器里。在一次 `docker run` 中可以挂载多个 `数据卷`。

下面创建一个名为 `web` 的容器，并加载一个 `数据卷` 到容器的 `/usr/share/nginx/html` 目录。

```bash
docker run -d -P --name web --mount  source=my-vol,target=/usr/share/nginx/html nginx:alpine
```

#### 查看数据卷的具体信息

在主机里使用以下命令可以查看 `web` 容器的信息

```bash
docker inspect web
```

```
"Mounts": [
    {
        "Type": "volume",
        "Name": "my-vol",
        "Source": "/var/lib/docker/volumes/my-vol/_data",
        "Destination": "/usr/share/nginx/html",
        "Driver": "local",
        "Mode": "z",
        "RW": true,
        "Propagation": ""
    }
],
```

#### 删除数据卷

```bash
docker volume rm my-vol
```

`数据卷` 是被设计用来持久化数据的，它的生命周期独立于容器，Docker 不会在容器被删除后自动删除 `数据卷`，并且也不存在垃圾回收这样的机制来处理没有任何容器引用的 `数据卷`。如果需要在删除容器的同时移除数据卷。可以在删除容器的时候使用 `docker rm -v` 这个命令。

无主的数据卷可能会占据很多空间，要清理请使用以下命令

```bash
docker volume prune
```

### 挂载主机目录

#### 挂载一个主机目录作为数据卷

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。

```bash
docker run -d -P --name web -v /data/hdb/docker_project/hub/webapp:/usr/share/nginx/html nginx:alpine 
```

以前使用 `-v` 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹，现在使用 `--mount` 参数时如果本地目录不存在，Docker 会报错。

~~~bash
docker run -d -P --name web --mount type=bind,source=/data/hdb/docker_project/hub/webapp,target=/usr/share/nginx/html nginx:alpine 
~~~

加了 `readonly` 之后，就挂载为 `只读` 了。如果你在容器内 `/usr/share/nginx/html` 目录新建文件，会显示如下错误

## Docker 中的网络功能

### 外部访问容器

容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 `-P` 或 `-p` 参数来指定端口映射。

```bash
docker run -d -P nginx:alpine
docker container ls -l
----------------------------------
CONTAINER ID   IMAGE                                     COMMAND                   CREATED          STATUS                PORTS                                     NAMES
d5e0bfe7ec29   nginx:alpine                              "/docker-entrypoint.…"   10 seconds ago   Up 9 seconds          0.0.0.0:32771->80/tcp, :::32771->80/tcp   competent_benz
```

使用 `docker container ls` 可以看到，本地主机的32771被映射到了容器的 80 端口。此时访问本机的 32768 端口即可访问容器内 NGINX 默认页面。

#### 映射所有接口地址

使用 `hostPort:containerPort` 格式本地的 80 端口映射到容器的 80 端口，可以执行

~~~bash
docker run -d -p 80:80 nginx:alpine
~~~

#### 映射到指定地址的指定端口

可以使用 `ip:hostPort:containerPort` 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1

```bash
docker run -d -p 127.0.0.1:80:80 nginx:alpine
```

用 `ip::containerPort` 绑定 localhost 的任意端口到容器的 80 端口，本地主机会自动分配一个端口。

```bash
docker run -d -p 127.0.0.1::80 nginx:alpine
```

还可以使用 `udp` 标记来指定 `udp` 端口

```bash
docker run -d -p 127.0.0.1:80:80/udp nginx:alpine
```

`-p` 标记可以多次使用来绑定多个端口

~~~bash
docker run -d -p 80:80 -p 443:443 nginx:alpine
~~~

### 容器互联

下面先创建一个新的 Docker 网络。

```bash
docker network create -d bridge my-net
```

`-d` 参数指定 Docker 网络类型，有 `bridge` `overlay`。其中 `overlay` 网络类型用于 [Swarm mode](https://docker-practice.github.io/zh-cn/swarm_mode)，在本小节中你可以忽略它。

#### 连接容器

运行一个容器并连接到新建的 `my-net` 网络

```bash
docker run -it --rm --name busybox1 --network my-net busybox sh
docker run -it --rm --name busybox2 --network my-net busybox sh
```

下面通过 `ping` 来证明 `busybox1` 容器和 `busybox2` 容器建立了互联关系。

在 `busybox1` 容器输入以下命令

~~~bash
ping busybox2
~~~

