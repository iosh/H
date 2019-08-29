---
title: 用 GitLab CI 进行持续集成
date: 2019-08-29 13:49:27
tags: javaScript
---

GitLab CI 使用笔记

<!-- more -->

# 简介

GitLab 自带了 GitLab Continuous Integration (GItLab CI),并且默认已经打开,要使用此项服务需要在项目的根目录下配置一个 `.gitlab-ci.yml` 配置文件描述 pipeline 工作流程,一般流程分为 `test` `build` `deploy`, 而 GItLab CI 使用 `GitLab Runner` 来执行 `pipeline` 中的每一步, 而这个 `GitLab Runner` 是需要安装在自己的服务器上的.

核心流程图如下

```
+--------------+                                                 +------------+
|              |                                                 |            |
| Commit/Merge +----->GItLab CI +------->GitLab Runner+--------> |Run pipeline|
|              |                                                 |            |
+--------------+                                                 +------------+

```

即 通过 Commit or Merge 触发 GitLab CI, 之后 GitLab CI 会通知已经注册好的 GItLab Runner 来执行 .gitlab-ci.yml 中配置的步骤

# 核心概念

## Pipeline

Pipeline 是一组分阶段执行的作业, 相当于一次构建任务, 例如一套完整的 Test Build Deploy, 代码库中的任何 Commit/Merge 都会触发 Pipeline 从而执行任务

## Stage

Stage 就是 Pipeline 中的每个步骤例如 Test Build 这种单个的步骤

```
+------------------------------------------------------------+
|                                                            |
|                                                            |
|    Pipeline                                                |
|                                                            |
|                                                            |
|  +------------+      +-----------+      +-------------+    |
|  | Stage Test +----->+Stage Build+----->+ Stage Deploy|    |
|  +------------+      +-----------+      +-------------+    |
|                                                            |
|                                                            |
|                                                            |
+------------------------------------------------------------+
```

在实际中一个 Pipeline 内可以包含若干个 Stage 上图只是示例

### Jobs

Jobs 就是单个 Stage 所执行的工作, 一个 Stage 可以有多个 Jobs, 这些 Jobs 是并行的,并且都执行成功之后当前的 Stage 才会成功,同理如果有一个 Jobs 执行失败了当前 Stage 就会失败,同理 Pipeline 就会失败

```
+----+Stage     +-----> Next Stage
|               |
|               |
+---> test 1+-->+
|               |
+---> test 2+-->+
|               |
+---> test 3 +--+

```

上面就是一些主要的概念,下面就是具体操作

# 安装

## 安装 GitLab Runner

安装 GitLab Runner 教程官网上有, 使用 Docker 进行安装. GitLab Runner 和 GitLab 版本不做强行要求,不过官网上说最好一样.

```yml
version: "3"

services:
  GitlabRunner:
    image: gitlab/gitlab-runner:latest
    restart: always
    volumes:
      - ./gitlab-runner/config:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
```

执行 `docker-compose up -d` 会生成一个 gitlab-runner 文件夹,文件夹下有一一个 config 文件夹, 文件夹下会有一个空的 config.toml 文件

## 注册 GitLab Runner

注册 GitLab Runner ,输入 以下命令进入容器内部进行注册, 打开自己的 GitLab 项目,在侧边栏最后一项的设置里面找到 CI/CD, 左边会显示注册用的 url 和 Token

```
docker exec -it 容器名称 gitlab-runner register
```

`Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):`

输入网页上的 URL

回车

`Please enter the gitlab-ci token for this runner:`

输入网页上显示的 Token

回车

`Please enter the gitlab-ci description for this runner`

输入这个 runner 的描述

`Please enter the gitlab-ci tags for this runner (comma separated):`

输入这个 runner 的标签

`Please enter the executor: parallels, shell, virtualbox, docker, docker-ssh, ssh, docker+machine, docker-ssh+machine, kubernetes, custom:`

使用了 docker 就输入 docker

`Please enter the default Docker image (e.g. ruby:2.6):`

输入默认使用的 docker 镜像 文档上写了输入 `alpine:latest`

这个时候在刷新网页就可以看到注册之后的 Gitlab Runner 已经连接的信息了

## gitlab-ci.yml

具体配置可以参照官网下面给出一个非常简单的

```yml
image: node:lts-alpine # 使用镜像

build:
  tags:
    - gitlabRunner # 这里是上面注册gitlab runner 时候输入的那个 tag 意思就是这个项目使用 指定的 gitlab runner 如果不写任务会被一直等待
  stage: build
  before_script:
    - yarn

  script: "yarn run build"
  only:
    - master
  cache:
    paths:
      - node_modules
```
