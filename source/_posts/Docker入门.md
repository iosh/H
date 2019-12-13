---
title: Docker入门
date: 2018-12-01 14:28:48
tags: Docker
categories: 
	- Docker
---

学习 Docker 使用入门

<!-- more -->

# 安装

安装没什么好说的, Docker 官网有详细的讲解,根据个人的系统来安装适合自己系统的 Docker 程序.

# 知识点

安装完成之后,需要简单了解的一些知识

## 概念

docker 有三个重要的概念分别是 `镜像(image)` `容器(container)` `仓库(repositor)`

这三个概念使用一个例子来进行描述:通过自己编写或者使用开源源码进行编译编译成一个系统,之后运行这个系统来完成一些任务,而且还可以将这些系统存放在云端进行分发.

1. 镜像(image)

通过上面的例子,可以清楚的了解到,编写一些源代码或者引用一些第三方库之后就拥有了一个完整项目的源代码.通过使用 docker 来编译这些源代码 docker 会生成一个 image .

2. 容器(container)

在上一步使用 docker 根据源文件生成了一个镜像(image),那么再通过 docker 来运行这个镜像,这里生成的运行环境叫做容器(container)

3. 仓库(repositor)

在拥有镜像之后,可以通过仓库来分发镜像,可以通过 docker 的 hub.docker.com 来分发镜像

三者个关系为: 通过源文件构建 镜像(image), 通过镜像(image) 生成容器(container), 还可以通过仓库来分发镜像

# 常用命令

## 获取镜像

```
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

在 docker hub 上有很多高质量官方维护的镜像可供使用,例如[node](https://hub.docker.com/_/node/)

docker 默认是从 docker hub 上拉去镜像的,可以使用`docker pull [Docker Registry 地址[:端口号]/]仓库名[:标签]` 来从自己需要的地方来拉去镜像

## 列出镜像

列出本地镜像

```
docker images ls

```

使用 images ls 命令来列出本地拉去的镜像,这条命令是不显示中间镜像的,中间镜像就是你制作的镜像所依赖的镜像,如果想查看本地所有镜像可以通过

```
docker images --all
或者使用简写
docker images -a
```

来列出本地所有的镜像

## 删除本地镜像

如果要删除本地镜像可以使用 `docker rmi [IMAGE]`来删除指定镜像,注意这里删除镜像需要先停止和删除容器,如果有使用这个镜像的容器存在的时候 docker 会给与警告 ⚠️,不推荐使用`docker rmi -f [IMAGE]`来强制删除镜像

## 删除容器

如果要删除本地生成的容器可以使用`docker rm [CONTAINER]`来删除容器,容器删除的时候如果容器正在运行需要停止容器使用`docker stop [CONTAINER]`来停止容器,使用`stop`是停止运行的指定容器,不推荐使用`docker kill [CONTAINER]`来`强行`停止容器.

# 使用 Dockerfile 来制作镜像

通过一个十分简单的 React 例子来制作一个镜像,并且通过这个镜像来运行容器显示出 React 的单页面网站.

1. 新建一个空的文件夹 app

使用`create-react-app`来创建一个 React 项目

`npx create-react-app app`

npx 是 npm 5 之后带的一个小工具,这个工具是帮助我们运行包的,简单的来说`npx [PACKAGENAME]` 执行这个命令之后,npx 这个工具会首先在`node_module`文件夹内寻找指定名称的包,如果找不到那么就会去 npm 上下载这个包并执行这个包(用完就删了一次性的).

之后使用编辑器打开 `app`目录

新建一个 `Dockerfile` 文件文件内写入

```Dockerfile

# 使用 node 10.14作为依赖镜像进行构建slim是node镜像的一个精简版不附带任何其他工具比较小,其他的可以看仓库介绍
# 这里使用了 docker 多阶段构建
FROM node:10.14-slim as build-stage

# 将当前目录的所有文件copy到镜像的app目录下
COPY . /app

# 指定接下来的工作目录为 app 文件夹
WORKDIR /app

# 运行命令
# 使用 npm安装yarn(这条命令可选使用npm还是yarn根据个人喜好)
RUN npm install -g yarn
# 使用yarn安装依赖 也可以使用 RUN npm install
RUN yarn


RUN yarn run build


```

之后在根目录新建一个 nginx.conf 文件,配置来自于[这里](https://github.com/SaraVieira/rick-morty-random-episode/blob/master/nginx.conf)

写入以下配置

```nginx

worker_processes 1;

events {
    worker_connections 8000;
    multi_accept on;
    use epoll;
}


http {

    server_tokens off;

    sendfile        on;
    tcp_nopush      on;

    tcp_nodelay     off;
# Enable Gzip compressed.
  gzip on;

  # Enable compression both for HTTP/1.0 and HTTP/1.1 (required for CloudFront).
  gzip_http_version  1.0;

  # Compression level (1-9).
  # 5 is a perfect compromise between size and cpu usage, offering about
  # 75% reduction for most ascii files (almost identical to level 9).
  gzip_comp_level    5;

  # Don't compress anything that's already small and unlikely to shrink much
  # if at all (the default is 20 bytes, which is bad as that usually leads to
  # larger files after gzipping).
  gzip_min_length    256;

  # Compress data even for clients that are connecting to us via proxies,
  # identified by the "Via" header (required for CloudFront).
  gzip_proxied       any;

  # Tell proxies to cache both the gzipped and regular version of a resource
  # whenever the client's Accept-Encoding capabilities header varies;
  # Avoids the issue where a non-gzip capable client (which is extremely rare
  # today) would display gibberish if their proxy gave them the gzipped version.
  gzip_vary          on;

  # Compress all output labeled with one of the following MIME-types.
  gzip_types
    application/atom+xml
    application/javascript
    application/json
    application/rss+xml
    application/vnd.ms-fontobject
    application/x-font-ttf
    application/x-web-app-manifest+json
    application/xhtml+xml
    application/xml
    font/opentype
    image/svg+xml
    image/x-icon
    text/css
    text/plain
    text/x-component;
  # text/html is always compressed by HttpGzipModule

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format compression '$remote_addr - $remote_user [$time_local] '
        '"$request" $status $upstream_addr '
        '"$http_referer" "$http_user_agent" "$gzip_ratio"';

    server {
        listen 80;
        access_log /var/log/nginx/access.log compression;

        root /var/www;
        index index.html index.htm;

        location ~* \.(?:manifest|appcache|html?|xml|json)$ {
            expires -1;
            # access_log logs/static.log; # I don't usually include a static log
        }

        location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to redirecting to index.html
            try_files $uri $uri/ /index.html;
        }

        # Media: images, icons, video, audio, HTC
        location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|htc)$ {
          expires 1M;
          access_log off;
          add_header Cache-Control "public";
        }

        location ~* \.(?:css|js)$ {
            try_files $uri =404;
            expires 1y;
            access_log off;
            add_header Cache-Control "public";
        }

        # Any route containing a file extension (e.g. /devicesfile.js)
        location ~ ^.+\..+$ {
            try_files $uri =404;
        }

        location /static/ {
            root /var/www;
        }
    }
}

```

之后在打开`Dockerfile`文件继续编写配置,完整文件如下

```Dockerfile
# 使用 node 10.14作为依赖镜像进行构建slim是node镜像的一个精简版不附带任何其他工具比较小,其他的可以看仓库介绍
# 这里使用了 docker 多阶段构建
FROM node:10.14-slim as build-stage

# 将当前目录的所有文件copy到镜像的app目录下
COPY . /app

# 指定接下来的工作目录为 app 文件夹
WORKDIR /app

# 运行命令
# 使用 npm安装yarn(这条命令可选使用npm还是yarn根据个人喜好)
RUN npm install -g yarn
# 使用yarn安装依赖 也可以使用 RUN npm install
RUN yarn


RUN yarn run build

# 第二阶段构建
FROM nginx:1.15

# copy 上面npm build出来的文件
COPY --from=build-stage /app/build/ /var/www

# copy nginx 配置文件
COPY --from=build-stage /app/nginx.conf /etc/nginx/nginx.conf

# 对外暴露端口
EXPOSE 80

# 容器运行时执行命令关闭进程守护
ENTRYPOINT [ "nginx", "-g", "daemon off;" ]
```

然后编写忽略文件,比如忽略`node_modules`

新建一个文件`.dockerignore`

写入以下内容

```
node_modules
.vscode
README.md
```

之后运行`docker build -t myapp .`来构建镜像(别忘了 . )

然后运行`docker run -p 3000:80 myapp`

打开浏览器访问 `http://localhost:3000/` 你就可以看到部署的网站了

# Dockerfile 文件常用指令

## FROM

FROM 指令有以下几种格式

```
FROM <image> [AS <name>]

FROM <image>[:<tag>] [AS <name>]

FROM <image><@digest> [AS <name>]

```

这个指令用于后续指令的基准镜像,FROM 必须是 Dockerfile 文件中的第一条命令

FROM 可以多次出现在文件中,用于多阶段构建,tag 和 digest 是可选的

## CMD

CMD 指令也有三种模式

```
CMD ["executable", "param1", "param2"] #首选模式
CMD ["param1", "param2"] #这会将这些参数传递给 ENTRYPOINT 作为参数
CMD command param1 param2
```

一个文件中只能有一个 CMD 命令,如果列出多个那么只有最后一个生效
CMD 命令的主要目的是为执行容器提供默认值(容器运行后就会运行 CMD 命令)

## EXPOSE

```
EXPOSE <port> [<port>/<protocol>...]
```

EXPOSE 指令通知 docker 容器在运行的时候监听指定的网络端口,可以指定监听 TCP 或者 UDP,如果未指定协议那么默认是 TCP

例如

```
ESPOSE 80/tcp
EXPOSE 80/udp
```

运行容器的时候可以通过使用`docker run -p 本机端口:docker EXPOSE端口`来进行端口映射

## ENV

```
ENV <key> <value>
ENV <key>=<value> ... #支持多条环境变量设置
```

EVV 将环境变量<key>设置为<value>

例如 node 常用的环境变量这样设置

```
ENV NODE_ENV="production"
```

## ADD

```
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] #包含空格的路径需要使用此项
```

ADD 指令用于从目录中复制文件,目录可以是当前工作目录或者远程的 URL,将文件复制到镜像的文件系统中

## COPY

```
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

COPY 和 ADD 指令相同都是向 image 中复制文件,不过没有 ADD 指令那么强,比如 ADD 可以指定远程 URL , 压缩文件等等,一般来讲使用 COPY 指令更多一些

## ENTRYPOINT

```
ENTRYPOINT ["executable", "param1", "param2"] # 首选模式
ENTRYPOINT command param1 param2
```

当指定了 ENTRYPOINT 后，CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令

就是可以通过命令行参数对 executable 附加参数,如果使用 CMD 的话就需要输入全称

## VOLUME

```
VOLUME["/data"]
```

VOLUME 指令用于创建具有名称的安装点,将其标记为从本机主机或者其他容器保存外部安装的卷,可以通过卷来共享目录

主机目录本质上是依赖于注释的,为了保持镜像的可移植行,因为不能保证目标主机所指定的目录都可用,因此无法从 Dockerfile 中指定目录,必须在创建或者运行容器的时候指定安装点

## USER

```
USER <user>[:<group>] or
USER <UID>[:<GID>]
```

USER 指令用于设置运行镜像任何设置的用户,例如使用 CMD RUN 这些指令的用户(比如有些指令不希望使用 root 用户权限进行运行),如果没有指定 USER 的话那么将使用 root 进行执行

## WORKDIR

```
WORKDIR /path/to/workdir
```

WORKDIR 指令用于设置其他指令的工作目录,例如 RUN ADD COPY 等

## AGR

```
ARG <name>[=<default value>]
```

ARG 指令定义了一个环境变量,可以使用`docker build --build-arg <varname>=<value>` 是该变量将传递给构建起,如果用户使用了未在 Dockerfile 文件中定义的构建参数则会发出警告

和 ENV 的区别是在容器运行时候 ARG 定义的变量是不存在的

## ONBUILD

```
ONBUILD [INSTRUCTION]
```

ONBUILD 指令用于当前镜像被另一个镜像作为依赖的时候才会运行,会在下一个构建的上下文进行运行,就像他已经在下游构建 FROM 指令之后插入在 Dockerfile 文件中.

任何指令都可以注册为 ONBUILD

## HEALTHCHECK

```
HEALTHCHECK [OPTIONS] CMD command #设置检查容器健康状况的命令
HEALTHCHECK NONE # 如果基础镜像有健康检查指令,这条指令可以屏蔽掉其他健康指令
```

## SHELL

```
SHELL ["executable", "parameters"]
```

SHELL 指令用于覆盖默认的 shell

# 数据管理

数据卷是一个可供一个或者多个容器使用的特殊目录,数据卷是保存 Docker 容器生成和使用数据的首选,它有以下优点:

1. 数据卷易于备份或者迁移
2. 可以使用 Docker CLI 命令或者 Docker API 管理卷
3. 数据卷可以跨系统
4. 可以在多个容器之间安全的共享数据卷
5. 数据卷驱动程序允许在远程主机或者云进行,还可以进行加密或者其他功能
6. 新创建的数据卷可以通过容器预先填充内容

可以使用 `-v`或者`--mount`

对于新用户应该使用--mount 因为比--value 语法更简单

[数据卷使用手册](https://docs.docker.com/storage/volumes/#populate-a-volume-using-a-container)
