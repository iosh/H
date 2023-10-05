---
title: Nginx
date: 2020-07-04 21:50:56
tags: Nginx
---

Nginx with lua

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
    server_name     zyx.ee
}
```

server_name 支持使用通配符正则表达式, 支持配置多域名, 服务名,当有多个 server 块时会存在匹配优先级的问题,优先级顺序如下:

1. 精确名字
2. 以 \* 开头的最长的通配符名称, 例如 \*.zyx.ee
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
    upstream  backend {}  # upstream 块,
    server 192.168.1.12:9000 weight=2;#weight 代表权重, 权重越高,则请求的比例越高
    server 192.168.1.23:9000 weight=3;


    server {
        listen  80;
        server_ame  zyx.ee; # 如果只有一个 server 的话可以不写
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

## 常见内置变量

Nginx 提供了一些用来获取 HTTP 和 TCP 的信息

| 变量名                   | 说明                                                                          |
| ------------------------ | ----------------------------------------------------------------------------- |
| \$arg_name               | 值 URL 请求中的参数,name 是参数的名字                                         |
| \$args                   | 代表 URL 中所有的请求的参数                                                   |
| \$binary_remote_addr     | 客户端地址以二进制数据的形式出现,通常回合一些限速模块一起使用                 |
| \$body_bytes_sent        | 发给客户端的字节数,不包含响应头                                               |
| \$bytes_sent             | 发给客户端的总字节数                                                          |
| \$document_uri           | 设置\$uri 的别名                                                              |
| \$hostname               | 运行 Nginx 的服务器名                                                         |
| \$http_referer           | 表示请求是从哪个页面链接过来的                                                |
| \$http_user_agent        | 客户端浏览器的相关信息                                                        |
| \$remote_addr            | 客户端 IP 地址                                                                |
| \$remote_port            | 客户端端口号                                                                  |
| \$remote_user            | 客户端名,通常在 author basic 模块中使用                                       |
| \$request_filename       | 请求的文件路径, 基于 root alias 指令和 URI 请求生成                           |
| \$request_time           | 请求被 Nginx 接收后,一只相应数据返回给客户端所使用的时间                      |
| \$request_uri            | 请求的 URI,带参数                                                             |
| \$request                | 记录请求的 URL 和 hTTP                                                        |
| \$request_length         | 请求的长度,包括请求行,请求头和正文                                            |
| \$server_name            | 虚拟主机的 server_name 值,通常是域名                                          |
| \$server_port            | 端口号                                                                        |
| \$server_addr            | 服务器的 IP 地址                                                              |
| \$request_method         | 请求的方式 例如 post get                                                      |
| \$schema                 | 请求协议, http https                                                          |
| \$sent_http_name         | 任意响应头,name 为响应头的名字,name 是小写                                    |
| \$realip_remote_addr     | 保留原来的客户地址,在 real_ip 中使用                                          |
| \$server_protocol        | 请求采用的协议名称和版本号                                                    |
| \$uri                    | 当前请求的 URI 在请求过程中 URI 可能会变化,例如内部重定向或使用索引文件的时候 |
| \$nginx_version          | Nginx 版本号                                                                  |
| \$pid                    | worker 进程的 PID                                                             |
| \$pipe                   | 如果请求是 HTTp 流水线 piplined 发送的 pipe 值为 p 否则为.                    |
| \$connection_requests    | 当前通过一个连接获得的请求数量                                                |
| \$cookie_name            | name 即 COokie 的名称, 可以得到 Cookie 的信息                                 |
| \$status                 | http 请求状态                                                                 |
| \$msec                   | 日志写入的时间,单位为秒,精度是毫秒                                            |
| \$time_local             | 在通用日志可是下的本地时间                                                    |
| \$upstream_addr          | 请求反向代理到后端服务器的 IP 地址                                            |
| \$ upstream_port         | 请求到反向代理后服务器的端口号                                                |
| \$upstream_response_time | 请求在后端服务器消耗的时间                                                    |
| \$upstream_status        | 请求在后端服务器的 HTTP 响应状态                                              |
| \$geoip_city             | 城市名称, 在 geoip 模块中使用                                                 |

## upstream 使用手册

利用 proxy_pass 可以将请求转发到后端服务器, 如果需要指向多台服务器就要用到 ngx_http_upstream_module ,它为反向代理提供了负载均衡以及故障转移等重要功能

代理多台服务器

```nginx
# 定义一组服务器

upstream test_servers{
    server 172.0.0.1:80 max_fails=5  fail_timeout=10s weight = 10;
    server  172.0.0.1:81 max_fails=5  fail_timeout=10s weight = 5;
    server 172.0.1.2 backup;
    server 172.0.0.3 down;
}

server {
    listen 80;
    location / {
        # 通过代理将请求发送给 upstream 命名的 HTTP 服务

        proxy_pass  http:/test_servers;
    }
}

```

指令 upstream
语法 upstream name {}
环境 http
含义 定义一组 htpp 服务器, 这些服务器可以监听不同的端口,以及 TCP 和 UNIX 套接字, 在以一个 upstream 中可以浑噩使用不同的端口 TCP 和 UNIX 套接字

# 常见模块

## 使用 ngx_http_headers_module 设置响应头

这是个默认自带的模块, 主要包含 add_header 和 expires 两个指令

1. expires 用法 expires 30d; expires epoch max | off 等 , 默认值为 off
   设置 Expires 和 Cache-Control 响应字段,主要用于控制缓存时间例如
   expires -1; # 输出响应头是 cache-control: no-cache, 表示不缓存
   expires 1h; # 输出的响应头是 cache-control: max-age=3600

2. add_header 用法 add_header Cache-Contron no-cache always;
   用来添加响应头字段. 最后的 always 字段表示在所有的响应中都加入这个响应头, Nginx 默认不会再 404 500 等状态码中添加响应头.

# 缓存系统

缓存在整个服务系统中都是极为重要的存在,既能提升请求的访问速度, 又可以减少后端请求的压力.

```nginx
# 设置缓存空间的名字及其存放路径和存放方式
proxy_cache_path  /data/nginxcahe levels=1:2  keys_zone=cachedata:100m  inactive=7d max_size=50g use_temp_path=off;

server {
    listen 80;
    location / {
        # 指定缓存空间和大小
        proxy_cache  cachedata;
        # 指定缓存的 HTTP 状态和缓存时间
        proxy_cache_valid   200  304  10s;
        proxy_cache_valid 301 302 100s;


        # 请求最少访问两次才会被缓存
        proxy_cache_min_uses  2;

        # 指定换粗的 key
        proxy_cache_key  $scheme$host$is_args$args;
        # 缓存 HTTP 请求方法类型
        proxy_cache_methods  GET HEAD;

        # 添加一个响应头, 用来标识请求是否命中缓存
        add_header  N-Cahe-Status  @upstream_cache_status;


        # 设置 on标识允许将请求的 HEAD 方法改成 GET 方法缓存;
        proxy_cache_convert_head  on;

        sendfile on;

        proxy_set_header  Host  $host:$server_port;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

        # 当缓存失效时,回去后端服务器获取数据, 然后,将返回给用户并缓存;
        proxy_pass http://192.168.1.1
    }
}
```

# lua

在 Nginx 中引入 lua 的几个原因:

1. Nginx 缺少 if elseif else 这种简单的逻辑, 通常需要使用很多 if,繁琐且可读性差
2. Nginx 缺少大小判断等表达式,导致实现一些简单的功能也比较复杂
3. Nginx 缺少动态限速功能, 如需相关规则生效需要重启 Nginx 且第三方和原生的配置都缺乏灵活性
4. Nginx 缺少冬天路由功能,如需要根据请求的路由动态调整服务器的转发规则则需要重启 Nginx
5. Nginx 在 API 网关系统中未实现智能化
6. Nginx 与数据库交互能理有限
7. 大多数 Nginx 模块都是使用 C 语言开发的, 且开发人员还需要了解 Nginx 内部构造,开发难度较大.

目前比较有名的 Nginx 框架有:
1. Tengine 由淘宝出品的 2011 年开源, 其由 Nginx + Lua 开发的优点凸显出了,并在淘宝网中拥有广泛事件
2. openRest 由国人章亦春发起, Tengine 和 openRest 区别在于 openRest 是完全开源产品与 Nginx 紧密配合, 而 Tengine 内部的 Nginx 更新会慢
