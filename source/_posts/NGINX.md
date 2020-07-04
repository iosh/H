---
title: Nginx
date: 2020-07-04 21:50:56
tags: Nginx
---

NGIXN with lua

<!-- more -->

Nginx 是一款高性能,高并发著称的 HTTP 服务器,主要用于反向代理,负载均衡,Web 网关等服务

Nginx 很适合作为 Nodejs 的前置服务器, 用来处理 Node 不适合处理的东西, 例如文件传输, Gzip 压缩, 等会长时间占用线程的操作. 这种操作都可以委托给 Nginx,而 Nodejs 仅仅用来处理请求, 从而发回最大特长.

# 基础配置

## Nginx 指令和指令块

```nginx
Main 1;
events {}
http {
    Main 2;
    server {
        Main 3;
        location {}
    };
    server {
        Main 3;
        location {}
    }
}
```

上面主要包含了两种指令:

- 简单指令, 由名称和参数组成, 以空格分隔, 以分号结尾, 例如上面的 Main 1; Main 2; 等
- 指令块: 由名称和 {} 组成,不以分号结尾, 例如 events http server location 等

上面例子中 http 是全局参数,针对整体产生影响, server 是对应的具体的服务,主要对指定的主机和端口进行配置, location 则是在具体服务下对 URI 进行配置, URI 即去掉参数的 URL

## 与客户端相关的配置

与客户端相关的配置主要在 http 块中进行, 下面是一些常见的指令

| 指令                         | 说明                                                                                 |
| ---------------------------- | ------------------------------------------------------------------------------------ |
| client_body_buffer_size      | 设置读取客户端请求体的缓冲区大小                                                     |
| client_body_temp_path        | 定义储存客户端请求体的临时文件目录,最多可以定义三个子集目录                          |
| client_body_timeout          | 定义读取客户端请求体的超时时间,即两个连续的读动作之间的时间间隔,超时抛出 408         |
| client_header_buffer_size    | 定义客户端请求头的缓冲区大小, 默认 1kb                                               |
| client_max_body_size         | 设置客户端请求的最大主体的大小, 默认 1Mb                                             |
| client_header_timeout        | 设置客户端请求头的超时时间                                                           |
| etag                         | 设置为 on 表示为静态资源自动生成 ETag 响应头                                         |
| large_client_header_buffer   | 设置大型客户端请求头的缓冲区大小                                                     |
| keepalive_timeout            | 设置连接超时时间,超过超时时间后会关闭 HTTP 连接                                      |
| send_timeout                 | 指定客户端的超时时间                                                                 |
| server_name_hash_bucket_size | 设置 server_name (Nginx 中配置的全部域名) 散列桶的大小, 默认取决于处理器缓存行的大小 |
| server_name_hash_max_size    | 设置 server_names 散列表的最大值                                                     |
| server_tokens                | 启用或者禁用在错误页面或者响应头中标识 Nginx 版本                                    |
| tcp_nodelay                  | 启用或者禁用 TCP_NODELAY 选项,只有保持当前活动时,才会被弃用                          |
| tcp_nopush                   | 仅当 sendfile 时使用, 能够将响应头和正文的开始部分一起发送                           |

## server 块

server 即服务部分, 如果请求头中的 Host 头和 server_name 匹配则将请求指向对应的 server 块

```nginx
server {
    server_name     huxiansheng.net
}
```

server_name 支持使用通配符正则表达式, 支持配置多域名, 服务名,当有多个 server 块时会存在匹配优先级的问题,优先级顺序如下:

1. 精确名字
2. 以 \* 开头的最长的通配符名称, 例如 \*.huxiansheng.net
3. 以 \* 结尾的最长通配符名称, 例如 huxiansheng.\*
4. 按照文件顺序,第一个匹配到的正则表达式
5. 如果没有匹配到对应的 server_name, 则会访问 default_server

## location 块

location 块在 server 块中使用, 他的作用是根据客户端请求的 URL 去定位不同的应用, 即当服务器接收到客户端请求之后, 需要在服务器端指定目录中寻找客户端所请求的资源

| 配置格式         | 作用                                                |
| ---------------- | --------------------------------------------------- |
| location=/uri    | 精确匹配                                            |
| location= ^~/uri | ^~ 匹配某个 URL 前缀开头的请求, 不支持正则表达式    |
| location ~       | ~ 区分大小写的匹配,属于正则表达式                   |
| location ~\*     | ~\* 标识不区分大小写匹配,属于正则表达式             |
| location /uri    | 标识前缀匹配, 不带修饰符,但是优先级没有正则表达式高 |
| location /       | 通用匹配,默认找不到其他匹配时,会进行匹配            |
| location @       | 命名空间,不提供常规的请求匹配                       |

匹配优先级也是按照上面的表格进行的, `@` 标识命名空间的位置,通常在重定向时进行匹配,且不会改变 URL 的原始请求

打开 debug 模式可以看到每个请求的执行过程,包括匹配到对应的 location 操作

有些指令可以在 location 块中执行, 主要有以下几个

- internal 表示该 location 只支持 Nginx 内部请求的访问, 例如支持 rewrite error_page 等重定向,但不能通过外部的 HTTP 直接访问
- limit_except 限定该 location 块可以执行的 HTTP 方法, 例如 GET
- alias 指定位置的替换
- root 也是指定位置的替换, 但是是相对位置 alias 是绝对地址

## include 的使用

include 用来指定主配置文件包含的其他扩展配置文件, include 可以出现在全局 location serve 等任何一个位置

include 还支持通配符导入例如:

```nginx
include  /home/test/*.config
```

## nginx 常见配置

```nginx
user www www; # 定义运行Nginx的用户和用户组
woker_processes2; # Nginx 进程数
woker_cpu_affinity  auto; # 配置Nginx 继承的 cpu 亲缘性
error_log  /var/log/error_log  info; # 定义全局的错误日志的类型默认是 error;
worker_rlimit_nofile  65535;  # 一个 worker 进程最多能打开的文件数量
pid /var/run/nginx.pid; # 进程文件
worker_priority  -10; # 在linux 系统下资源使用的优先级
worker_shutdown_timeout  30; # 若30s无法平滑退出,则强行关闭进程

events {
    # 单个进程的最大连接数, 整个 Nginx 的最大连接数 = 单个进程的最大连接数 * 进程数
    worker_connections  10000;

    # nginx 网络模型 epoll 用在 linux 2.6 + 的高性能的网络 I/O上, 如果是在 FreeBSD 上, 则用 kqueue 模型
    use epoll;
}

http {
    include  conf/mime.types;   # 引入文件扩展名与文件类型的映射表;
    default_type  application/octet-stream; # 默认文件类型
    log_format main '$remote_addr - $remote_user [$time_local]'
                    ' "$request" $status $bytes_sent'
                    ' "$http_referer" $ "$http_user_agent"'
                    '"$htt_cookie"'; # 自定义日志格式

    client_header_buffer_size  1k; # 设置用户请求头使用的 buffer 的大小
    large_client_header_buffers 4 4k; # 默认的缓冲区大小不够用时就会使用此参数
    server_names_hash_bucket_size  128;#设置 server_names 散列桶的大小

    gzip on; # 设置 gzip
    gzip _comp_level 6; # 设置压缩等级
    gzip _min_length 1100; # 设置允许压缩的页面最小字节数
    gzip _buffers   4 8k; # 设置系统需要获取多大缓存用于储存 gzip 的压缩结果胡拘留 48看代表按照原始数据的大小, 即以 8kb 为单位的 四倍申请内存
    gzip _types  text/plain  text/css ; # 匹配MIME类型压缩
    output_buffers  2  32k  # 色值用于从磁盘读取响应的缓冲区的数量和大小 2  32k 代表按照原始数据 即 32k 为单位的 2倍申请内存
    sendfile on; # 启用sendfile()函数
    tcp_nopush on; # #为了防止网络堵塞,需要开启 sendfile
    tcp_nodelay on; #为了防止网络堵塞需要开启 sendfile
    keepalive_timeout  90s; # 长连接超时时间, 单位是秒
    upstream  backend {}  # upstream 块, weight 代表权重, 权重越高,则请求的比例越高
    server 192.168.1.12:9000 weight=2;
    server 192.168.1.23:9000 weight=3;


    server {
        listen  80;
        server_ame  huxiansheng.net; # 如果只有一个 server 的话可以不写
        access_log  /var/log/nginx.access_log   main; # 访问日志记录
        charset utf-8   # 默认编码

        location / {
            proxy_pass  http://localhost:8000; # 请求转发
            proxy_redirece  off; # 如果需要修改从被代理服务器传来的应答头中的"Location"和"Refresh"字段，可以用这个指令设置。
            proxy_set_header Host   $host;
            proxy_set_header X-Real-ip  $remote_addr;
            proxy_set_header X-Forwarded_For $proxy_add_x_frwarded_for;
        }

        error_page  404  /404.html;# 对后端服务器抛出的 404页面进行重定向
        location /404.html {
            root /spool/www
        }


        location ~*\.(jpg|jpeg|gif)$ {
            root  /spool/www;
            expires  30d; #浏览器保留缓存时间
        }

    }
 }


```
