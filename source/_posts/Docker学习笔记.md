---
title: Docker学习笔记
date: 2018-04-21 19:30:32
tags: Docker
---

Docker 实践学习笔记

<!-- more -->

Docker  是相当了不起的，当它未出现之前，应用程序还是一些庞大的单体软件，独自运行在一堆钢铁和硅块上，持续多年，并且拒绝改变，不思进取，对于想要快速前行的组织而言，这的确是一个难题，因此虚拟机的崛起也就不足为奇，应用程序不必再和这些硬件捆绑在一起，使得这一切可以更快更替，更加灵活。

但是虚拟机本身也是很复杂的，那么应该如何管理所有的复杂性，通过配置管理当然可以办到，但是配置管理就不复杂了吗。

而我们的 Docker 采取了一种截然不同的做法，如果用户将软件放在一个容器内，他会将应用程序本身的复杂度和基础设施`相隔`开来，这使得基础设施本身变得更加简单，而应用程序也更加易于装配，基于此，与传统虚拟机相比，计数速度和执行效率都有了巨大的飞跃，容器的启动也成为了毫秒级，而不是分钟级，内存是共享的，而不是预先分配，这使得应用程序能够以更低的成本运行，同时也意味着可以按照想要的方式架设应用，而不在收缓慢的、不太领过的基础设施的制约。

Docker 在关键的适合出现了，正是为了迎合许多软件祖师的一个迫切需求：以一种开放和灵活的方式构建软件，然后在不同的环境下能够可靠的一致的部署它，用户不需要学习新的编程语言，购买昂贵的硬件，也无需在为了构建、部署和运行可抑制的应用程序而在安装或者配置过程上花费大量时间。

# Docker 基础

## Docker 初探

Docker 是一个允许用户`在任何地方构建、分发及运行任何应用`的平台。

Docker 的出现，改变了开发中用于管理软件活动的不同技术组合而成的状态，因为这些工具需要由专业工程师管理和维护，而且多数工具具有自己`独特`的配置方式。Docker 出现允许不同的工程师参与到这个过程中，有效的使用一门语言，这让协作变得轻而易举，所有东西通过共同的流水线转变成可以在任何目标平台使用的单一产物，无需继续维护一堆让人眼花缭乱的工具配置。

## Docker 是什么

要理解 Docker 是什么，从一个比喻开始会比计数性解释来的更简单，而且这个 Docker 的比喻非常具有说服力，Docker 原本是指在船只停靠在港口之后将商品转移或移出的工人，箱子和物品的大小和形状各异，而有经验的码头工人能以核算的方式将手工将商品装入船只，因而他们备受青睐，雇人搬东西并不便宜，但除此之外别无选择。

对于软件行业工作的人来讲，这听起来应该很熟悉，大量时间和精力将被花费在各种奇形怪状的软件防止到装满了其他奇形怪状软件、大小各异的船只上、以便将其卖给其他地方的用户或商业机构。

在 Docker 出现之前，部署软件到不同的环境所需工作量巨大，即使不采用手工运行脚本的方式在不同的机器上进行软件配置，用户也不得不权利应付哪些配置管理工具，他们掌握着渴求资源且快速变化的环境的状态，即便将这些工作封装到虚拟机中，还是需要花费大量时间来部署这些虚拟机，等待他们恩启动并管理他们所产生的额外的资源开销。

使用 Docker ，配置工作从资源管理中分离出来，而部署工作则是微不足道的：运行 docker run ，环境的镜像会被拉去下来并准备运行，所消耗的资源更少并且是内含的，因此不会干扰到其他环境。

而使用者则不需要担心容器是如何被分发到任何机器中，只要上面右 Docker ，那么一切都不是问题。

## Docker 有什么好处

几个重要的问题，出现了`为什么要使用 Docker`,`Docker 用在什么地方`，针对为什么的简要答案是：只需要一点点付出，Docker 就能快速为企业节省大量时间和金钱。

1. 代替虚拟机 VM

   Docker 可以在很多情况下替代虚拟机，如果用户只关心应用程序，而不是操作系统，可以使用 Docker 替代虚拟机，并将操作系统交给其他人考虑，Docker 不但启动速度快，迁移的同时也更为轻量，同时得益于他的分层文件系统，与其他人分享时候变得更简单，更快捷。而且它牢牢的扎根在命令行中，非常适合脚本化。

2. 软件原型

   如果想快速体验软件，同时为了避免干扰目前的设置或者配备一个虚拟机的麻烦，Docker 可以在毫秒级内提供一个沙箱环境。

3. 打包软件

   因为对 Linux 用户而言，Docker 镜像并没有依赖，所以非常适合用来打包软件，用户可以构建镜像，并确保它可以运行在任何 Linux 机器上。

4. 让微服务构架成为可能

   Docker 有助于将一个复杂系统分解成一系列可组合的部分，这让用户可以用更离散的方式来思考其服务，用户可以在不影响全局的前提下重组软件使其各部分更容易管理和可插拔。

5. 网络建模

   由于可以在一台机器上启动数百个甚至数千个隔离的容器，因此对网络进行建模轻而易举，对于实现现实世界的测试场景非常有用，而且所费无几。

6. 离线时启用全栈生产力

   因为可以将系统中所有部分捆绑在 Docker 容器中，用户可以将其编排运行在笔记本电脑中移动办公，即便在离线时也没什么问题。

7. 降低调试支出

   大家想必都遇到过，代码在进行交互的时候，想在一个新的电脑上跑起来，异常麻烦。真的很麻烦，还说不定到处报错，甚至无法启动，而使用 Docker 可以让用户清晰的说明即便是脚本的形式，在一个属性已知的系统上调试的问题，错误和环境重现变得简单，这得益于 Docker 与提供宿主环境的机器，是分离的。

8. 文档化软件依赖及接触点

   通过文档化结构方式构建镜像，为迁移到不同环境做好准备，Docker 强制用户从一个基准点开始明确记录软件依赖，即使用户不打算在所有地方都使用 Docker，这种文档记录需要也有助于在其他地方安装软件。

9. 启用持续交付

   持续交付是一种基于流水线的软件交付类型，该流水线通过一个自动化或者半自动化流程在每次变动时重新构建系统然后交付到生产环境中。

   因为用户可以更准确的控制构建环境的状态，Docker 构建比传统软件构建方法更具有可重现性和复制性，是持续交付的实现变得更容易，通过一个以 Docker 为中心的可重现的构建过程，标准的持续交付计数，变得很简单。

## Docker 关键的命令

Docker 的中心功能是构建、分发及在任何具有 Docker 的地方运行软件，对于一个终端用户而言，Docker 是一个用于运行命令行的程序，就像 git 一样，这个程序具有执行不同操作的子命令。

Docker 的子命令

|     命令      |                目的                |
| :-----------: | :--------------------------------: |
| docker build  |         构建一个Docker镜像         |
|  docker run   |  以容器的形式运行一个 Docker 镜像  |
| docker commit | 将一个 Docker 容器作为一个镜像提交 |
|  docker tag   |      给一个 Docker 镜像打标签      |

## 镜像与容器

如果不熟悉 Docker ，可能是第一次听说上面的`容器`和`镜像`这两个词语，它们是 Docker 中最重要的概念，因此需要花点时间明确其中的差异。

看待`镜像`和`容器`的方式是将他们类比为`程序`和`进程`，一个`进程`可以视为一个`被执行的应用程序`，同样，一个 Docker 容器可以视为一个运行中的 Docker 镜像。

如果熟悉`面向对象`，看待镜像和容器的另一种方法是`将镜`像看做`类`，而将`容器`看为`对象`，`对象`是`类`的具体实例，同样，`容器`是`镜像`的实例，用户可以从`单个镜像`创建`多个容器`，就像`对象`一样，他们之间是`相互隔离`的，无论用户在对象内修改了什么，`都不会影响`到`类`的定义，因为他们从本质上根本就是`不同的东西`。

> 这里穿插一下 Docker 的基础知识，因为我也不会 Docker ，而本书并不会讲 Docker 如何安装运行等

## Docker 安装与运行

我是看的阮一峰老师写的[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到。下面的介绍都针对社区版。

Docker CE 下载地址

- [Mac](https://docs.docker.com/docker-for-mac/install/)
- [Windows](https://docs.docker.com/docker-for-windows/install/)  请注意 windows 安装比较麻烦，需要系统开启 Hyper-V 功能，具体怎么做百度有，系统版本需要是Windows 10 企业版、专业版或教育版 才有这个功能，那么家庭版没这个功能怎么办，请前往淘宝寻找解决方案
- [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
- [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
- [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
- [其他 Linux 发行版](https://docs.docker.com/install/linux/docker-ce/binaries/)

我目前使用的是 Ubuntu 系统

首先更新软件源

```
sudo apt-get update
```
更新以下软件，允许通过HTTPS使用储存库

```
sudo apt-get install apt-transport-https  ca-certificates  curl  software-properties-common
```

添加 Docker 官方的 GPG 秘钥

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
官方的秘钥指纹是

```
9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
```

通过搜索秘钥指纹后八位来确定是否正确，次步骤是为了安全起见，可能吧

```
sudo apt-key fingerprint 0EBFCD88
```

在命令行输入

```
lsb_release -cs
```

看它的返回值是什么来分别安装我的返回值是 `xenial` 代表 `Ubuntu 16.4` 不知道这一步有什么意义

基本上个人电脑和服务器电脑都是 x86 构架输入一下命令

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

这一步应该是将下载地址指向正确的系统构架地址

再次更新 `apt`软件包索引，以便可以搜到 Docker

```
sudo apt-get update
```

安装最新版的 Docker CE

友情提示，带梯子全局代理之后输入下面的命令，不然根本下载不动。至于如何用梯子全局代理，百度，如何不使用梯子下载 Docker ，百度。不在废话。

```
sudo apt-get install docker-ce
```

测试是否安装完毕在命令行输入

```
docker version

之后输出类似如下内容表示 Docker 已经安装完成
Client:
 Version:	18.03.0-ce
 API version:	1.37
 Go version:	go1.9.4
 Git commit:	0520e24
 Built:	Wed Mar 21 23:10:01 2018
 OS/Arch:	linux/amd64
 Experimental:	false
 Orchestrator:	swarm

```

Docker 需要用户具有 root 权限，为了避免每次命令都输入 sudo，可以将用户加入 Docker 用户组

创建 Docker 用户组

```
sudo groupadd docker
```

提示 docker 组已存在，应该是安装的时候创建了

将用户添加到 Docker 用户组中

```
sudo usermod -aG docker $USER
```

之后重启或者注销，才会生效

之后可以输入

```
docker run hello-world
```

输入这条命令之后，会有两个情况

情况一报错

```
docker: Get https://registry-1.docker.io/v2/library/hello-world/manifests/latest: net/http: TLS....
```

遇到这个问题，就是不可描述问题，一全局梯子，二换源

换源有两个推荐

[Docker 中国官方镜像](https://www.docker-cn.com/registry-mirror)

[daocloud加速镜像](https://www.daocloud.io/mirror#accelerator-doc)需要注册

情况二就是正常输入欢迎信息。

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
.......
```

输出这段欢迎词之后容器就会自动终止，但是有些是提供服务的，例如安装和运行 Ubuntu 的 image ，就可以在命令行体验 Ubuntu 系统

对于不会自动终止的容器，必须使用

```
docker container kill [containID]
```

id 可以通过

```
docker image ls
```

找到对应的id，结束掉进程

## 定义一个镜像并运行容器

接下来使用`node`框架`express`在3000端口显示一个hello world并且制作成 Docker 的镜像

新建一个demo文件

demo作为 Docker 的工程目录

```JavaScript
将终端路径切换到demo下输入
npm init // 创建一个package文件
npm i express // 安装express
```

然后在新建一个 index.js 文件然后里面写上 espress 标准的 hello world 代码

```JavaScript
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

var server = app.listen(3000, function () {
  var host = server.address().address;
  var port = server.address().port;

  console.log('Example app listening at http://%s:%s', host, port);
});
```

那么现在有了一个 demo 文件夹，文件夹内有

`index.js`  `node_modules`  `package.json`  `package-lock.json`

index.js 文件里面我们写了上面的代码，而且可以通过 node index.js 来运行这段代码 确定启动服务，http://localhost:3000/ 端口也可以显示 hello world！

接下来就是将制作 Docker 镜像

现在，在 demo 文件夹中新建一个文件，文件名为 `.dockerignore`这个文件类似于 git 的忽略文件，之后在其中写入

```dockerfile
.git
node_modules
npm-debug.log
就是忽略这三个文件和文件夹
```

接下来新建另一个文件`Dockerfile`这个就是 Docker 工作文件，Docker 会根据文件内的内容一步一步执行，在文件中写入如下内容

```dockerfile
# 将官方的 node 8.11.1
FROM node:8.11.1
# 将工作目录设置为 /app
WORKDIR /app
# 将当前目录下的(demo目录)app文件夹复制到/app中的镜像中
ADD . /app
# 运行npm 安装以来包
RUN npm install --registry=https://registry.npm.taobao.org
# 对外暴露 3000 端口
EXPOSE 3000
# 容器启动后运行该条命令
CMD node index.js
```

那么 demo 文件夹内现在有以下文件

  `index.j` express源代码
  `node_modules` 依赖文件夹（上面写了忽略规则，所以会被忽略
  `package.json`npm的工作文件
  `Dockerfile` Docker 工作文件
  `.dockerignore`  Docker构建镜像忽略文件

接下来构建 Docker 镜像及 image 文件

```Docker
# 使用 Docker 生成 image 文件， -t 是用来指定 image 文件的名字 我起了 demoapp ，如果不指定那么默认标签就是 latest 最后的点代表 Dockerfile 文件所在位置 当前路径就是 .
docker build -t demoapp .
```

接下来如果没有报错，那么就是等待 Docker 依据 Dockerfile 文件下载依赖，有点慢，换了国内镜像还是慢，不知道是不是我网速问题。

成功之后

```
# docker 以 demoapp 为镜像运行一个容器，映射主机5000端口和容器的3000 端口，运行完毕之后容器不会自动删除
docker container run -p 5000:3000 -it demoapp

# 和上面的命令一样但是运行后会自动删除容器 可以使用 docker container ls --all 查看容器是否被删除了
docker container run --rm -p 5000:3000 -it demoapp
```

之后就会用浏览器打开

```
http://localhost:5000
```

可以访问到并且反返回了 hello world 那么就成功了

```
docker container run 
# 上面这条命令每运行一次都会生成一个容器，如果不想生成新的容器应该使用下面的命令

docker container start 容器id

# 查看容器ID命令使用 docker container ls --all
```

以下是上面所有的 Docker 用到或者可以用到的一些相关命令

```do
docker build -t friendlyname .     # 使用此目录的 Dockerfile 创建镜像（命令最后有个点别忘了
docker run -d -p 4000:80 friendlyname         # 运行镜像，后台模式
docker ps           # 查看所有正在运行的容器的列表 或者使用 docker container ls --all
docker stop <CONTAINER ID>                     # 安全结束容器
docker ps -a           # 查看所有容器的列表，甚至包含未运行的容器
docker kill <CONTAINER ID>                   # 强制关闭指定的容器
docker rm <CONTAINER ID>              # 从此机器中删除指定的容器
docker rm $(docker ps -a -q)           # 从此机器中删除所有容器
docker images -a                               # 显示此机器上的所有镜像
docker rmi <imagename>            # 从此机器中删除指定的镜像
docker rmi $(docker images -q)             # 从此机器中删除所有镜像
docker login             # 使用您的 Docker 凭证登录此 CLI 会话
docker tag <image> username/repository:tag  # 标记 <image> 以上传到镜像库
docker push username/repository:tag            # 将已标记的镜像上传到镜像库
docker run username/repository:tag                   # 运行镜像库中的镜像
```

之后使用上面的命令将本地的`镜像`和`容器`都删掉吧，感觉还是满占地方的

```
docker rm $(docker ps -a -q) # 删除所有容器
docker rmi $(docker images -q) # 删除所有镜像
```

## Docker 微服务教程

这里 还是阮老师的教程 [教程地址](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

看完之后然后我在准备看一下官方文档，之后继续回去看书。

因为阮老师的教程写的很详细，我就不会照抄阮老师的教程了，我只会将我觉得比较重要的东西记录下来

阮老师教程上使用了三种方法，演示如何假设 WordPress 网站，分别动手试试

- 方法 A：自建 WordPress 容器
- 方法 B：采用官方的 WordPress 容器
- 方法 C：采用 Docker Compose 工具

第一种方法是`事无巨细，皆需亲为`，从开始搭建一遍

第二种方法是`使用官方提供的便捷镜像`，快速搭建服务

第三种方法是`使用 Docker 官方容器管理工具`方便管理多个容器互相协作

一个一个动手写一下

### 自建 WordPress 容器

首先新建一个`demo`文件夹，并且将命令行目录切换到`demo`文件夹，手动或者命令行命令都可以

```
mkdir demo && cd demo
```

然后使用命令创建一个基于`php`的 image (镜像) 新建一个容器，并且运行这个容器，`php`的标签是`5.6-apache`说明 `php`版本是5.6并且自带了`Apache`服务器

```
docker container run \ # Docker 运行一个容器，
  --rm \ # 运行结束后删除容器
  --name wordpress \ #容器的名字叫做 wordpress
  --volume "$PWD/":/var/www/html \ # 将当前目录（demo）映射到容器的 /var/www/html 因为 Apache 对外访问的默认目录是这个，因此在 demo 文件中的任何修改，都会反应到容器里面，从而被外部访问到
  php:5.6-apache # 指定镜像名称

  # 这是一行命令， \ 带便换行符，将一条命令分成几行展示，和下面这条命令是一样的,复制下面的命令，我在上面命令后面写了注释

docker container run --rm --name wordpress  --volume "$PWD/":/var/www/html php:5.6-apache
```

运行上面的命令之后，Docker 就会去镜像仓库下载镜像，并根据镜像生成名为 wordpress 镜像。然后服务就跑起来了，简单吧。

那么emmmmm不出不知道你们是否可以访问到`172.17.0.2`我访问不到，那么怎么办呢，ctrl + c 结束掉当前容器(他会自己删除容器),之后输入

```
docker container run -p 80:80   --rm   --name wordpress   --volume "$PWD/":/var/www/html   php:5.6-apache
```

多了一个端口暴露指令 `-p 80:80`

之后访问

http://localhost/

就可以访问啦，然后就看到

> Forbidden  You don't have permission to access / on this server.

我们已经映射了容器的`/var/www/html` 到 `demo`文件夹下，当前因为`demo`文件夹没有文件，所以无法提供访问，添加一个最简单的 php 文件，在 demo 文件夹下新建一个 index.php 文件，在文件内写入

```php
<?php 
phpinfo();
?>
```

之后在访问

http://localhost/

就可以访问到一个页面了

接下来从 [WordPress 官方网站](https://cn.wordpress.org/) 中下载下来安装包,或者直接点击 [下载WordPress4.9.4安装包](https://cn.wordpress.org/wordpress-4.9.4-zh_CN.zip)

之后将 WordPress 压缩包里面的 WordPress 文件夹放到 demo 文件夹内然后访问

http://localhost/wordpress/

就可以看到安装界面了，那么要安装和运行 WordPress 还需要一个数据库 MySQL 数据库，那就再跑一个镜像，这个镜像跑 MySQL。

这样逻辑上就是跑两个容器 一个 WordPress 容器， 一个 MySQL 容器

新建一个命令行窗口，刚才哪个别关里面还跑着我们的 WordPress 呢

```
docker container run \ # Docker 运行一个容器
  -d \ # 容器启动后，在后台运行
  --rm \ # 运行结束后删除容器
  --name wordpressdb \ # 容器名称为 wordpressdb
  --env MYSQL_ROOT_PASSWORD=123456 \ # 向容器传进一个环境变量 MYSQL_ROOT_PASSWORD，等号后面的值会作为 MySQL 的根密码，不太懂这个根密码什么意思
  --env MYSQL_DATABASE=wordpress \ # 向容器传进一个环境变量 MYSQL_DATABASE，等号后面的值会被创建同名的数据库
  mysql:5.7 # MySQL数据库镜像

  # 上面的还是一条指令为了方便写注释就拆成多行，直接复制下面这行就可以了
docker container run -d --rm --name wordpressdb --env MYSQL_ROOT_PASSWORD=123456 --env MYSQL_DATABASE=wordpress mysql:5.7
```

上面这条命令多了一个 -d 容器运行后，会自动后台，可以使用

  docker container ls

开看看当前运行的容器

```
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                NAMES
56da209af68b        mysql:5.7           "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp             wordpressdb
cd8b32dc65ae        php:5.6-apache      "docker-php-entrypoi…"   23 minutes ago       Up 23 minutes       0.0.0.0:80->80/tcp   wordpress

```

会发现数据库容器确实在运行中，数据库是后台运行的，前台看不到他的输出，如果像看到它的输出，必须使用下面的命令

  docker container logs wordpressdb

接下来需要将 WordPress 和 MySQL 链接起来，但是官方的 PHP 镜像不带 MySQL 扩展，所以得自己新建 镜像

首先 ctrl + c 退出 WordPress 容器进程，因为在启动命令中右 `--rm` 所以退出后容器会自己删除。

之后在 demo 目录中新建一个 `Dockerfile` 文件，并写入下面的内容

```dockerfile
# 指定容器镜像为php:5.6-apache
FROM php:5.6-apache
# 运行php官方提供的安装扩展指令，安装mysqli插件，方便链接MySQL数据库
RUN docker-php-ext-install mysqli
# 之后运行 apache2-foreground 命令跑起来 php环境
CMD [ "apache2-foreground" ]
```

接下来命令（demo文件路径下）

```
# 基于我们上面写的 Dockerfile 文件生成一个镜像 phpwithmysql 别忘了，最后还有个点，然后等待生成镜像

docker build -t phpwithmysql .

# 修改 demo 文件夹下 wordpress 文件夹的权限，保证它有写入权限，以便安装

chmod -R 777 wordpress

# 接下来根据镜像新建一个 WordPress 容器

docker container run \ # 运行一个容器
  --rm \ # 结束后删除容器
  -p 80:80 \ # 映射本地80端口到容器的80端口
  --name wordpress \ # 容器名称为 WordPress
  --volume "$PWD/":/var/www/html \ # 将demo 文件夹映射到容器的 /var/www/html 目录
  --link wordpressdb:mysql \ # 链接 wordpress 和 mysql 数据库
  phpwithmysql # 依据 phpwithmysql 镜像生成容器

# 直接复制下面的命令就可以了

docker container run -p 80:80  --rm   --name wordpress   --volume "$PWD/":/var/www/html   --link wordpressdb:mysql   phpwithmysql

```

接下来访问

http://localhost/wordpress/

就会跳转到 WordPress 的安装界面，点击现在就开始按钮进行安装， 接着 WordPress 要求数据数据库参数，输入参数如下

`数据库名`： wordpress

`用户名`： root

`密码`： 123456

`数据库主机`： mysql

`表前缀`： wp_

接下来就不用说了。之后就成了，也就安装完毕了。第一步也就完成了接下来关闭容器（他会自动删除容器），删除我们生成的容器。

```
# 停止php 和 MySQL 容器 他会自动删除 因为启动的时候添加了 --rm
docker container stop wordpress wordpressdb

# 然后列出本地的 镜像文件 吧用不着的都删除了
docker image ls

# 然后根据列表上面的id 删除镜像，此步骤可选，想删除就删除不想就留着
docker rmi 镜像id  镜像id  镜像id

# 可以连续删除多个本地镜像
```

### 采用官方的 WordPress 容器

上面采用了自己构建 WordPress 容器，分别生成 php 容器和 MySQL 容器，然后还需要链接起来，很麻烦，Docker 已经提供了官方的 WordPress image 镜像文件，直接用就可以了。

[这里有镜像的详细使用说明都可以看看](https://hub.docker.com/_/wordpress/)  里面有启动变量参数

首先新建并启动 MySQL 容器

```
docker container run \
  -d \ # 后台运行
  --rm \ # 结束后删除容器
  --name wordpressdb \ # 容器名称为 wordpressdb
  --env MYSQL_ROOT_PASSWORD=123456 \ # 根密码
  --env MYSQL_DATABASE=wordpress \ # 数据库名称
  mysql:5.7
```

之后就会在后台运行 MySQL 服务

然后运行基于官方的 WordPress image ，新建并启动 WordPress 容器

```
docker container run \
  -d \ # 后台运行
  -p 80:80 \ # 映射本机和容器端口
  --rm \ # 结束后自动删除容器
  --name wordpress \ # 容器名称
  --env WORDPRESS_DB_PASSWORD=123456 \ # 通过变量向容器内传入数据库密码
  --link wordpressdb:mysql \ # 链接数据库 和 WordPress 两个容器
  wordpress # 镜像名称

  # 老规矩复制下面的命令
  docker container run -d -p 80:80 --rm --name wordpress --env WORDPRESS_DB_PASSWORD=123456 --link wordpressdb:mysql wordpress
```

之后访问就可以看到安装界面了

http://localhost/

不想玩了就输入

```
docker stop wordpress wordpressdb
```

关闭两个容器进程就可以了

### 采用 Docker Compose 工具

Docker Compose 是用来提供一种更简单的方法来管理容器之间的联动，就不用我们一个一个去启动还要传参数。

windows 和 Mac 安装 Docker 同时一同安装Docker Compose ，linux 安装 [Docker Compose 教程](https://docs.docker.com/compose/install/#prerequisites) 

安装完毕之后输入

  docker-compose --version

验证是否安装成功

之后在 demo 文件夹中，新建 docker-compose.yml 文件写入下面内容

```
# mysql 容器配置
mysql:
    # 容器的镜像
    image: mysql:5.7
    # 环境变量分别输入 数据库根密码 和 数据库名称
    environment:
     - MYSQL_ROOT_PASSWORD=123456
     - MYSQL_DATABASE=wordpress
# WordPress 容器配置
web:
    # 容器镜像
    image: wordpress
    # 链接到 MySQL 容器
    links:
     - mysql
    # 输入环境变量，就是数据库的密码
    environment:
     - WORDPRESS_DB_PASSWORD=123456
    # 映射端口本地和容器端口
    ports:
     - "80:80"
    # 映射容器目录到
    working_dir: /var/www/html
    # 设置容器工作目录
    volumes:
     - wordpress:/var/www/html
```

```
# 启动所有服务
$ docker-compose up
# 关闭所有服务
$ docker-compose stop
```

使用启动命令之后就可以打开 

http://localhost/

进行访问了