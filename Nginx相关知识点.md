#   Nginx相关知识点 

## Nginx基础

### Nginx简介

#### Nginx出现的背景介绍

Nginx("engine x")是一个具有高性能的【HTTP】和【反向代理】的【web服务器】，同事也是一个【POP3/SMTP/IMAP代理服务器】,由伊戈尔 塞索耶夫使用C语言编写的，Nginx的第一版本是在2004年10月4号发布的0.1.0版本，并将Nginx源码进行了开源，使Nginx得到了一个良好的发展

#### 名词解释

1. web服务器
   1. web服务器也叫网页服务器，英文名Web Server，主要功能是为用户提供网上信息浏览服务
2. HTTP
   1. HTTP是超文本传输协议的缩写，是用于从WEB服务器传输超文本到本地浏览器的传输协议，也是互联网上应用最广泛的一种协议。HTTP是一个客户端和服务端请求和应答的标准，客户端是终端客户，服务端是网站，通过使用web浏览器，网络爬虫或者其它工具。客户端发起一个到服务器上指定端口的HTTP请求
3. POP3/SMTP/IMAP
   - POP3(Post Office 3) 邮局协议的第三个版本
   - SMTP(Simple Mail Transfer Protocol) 简单邮局传输协议
   - IMAP(Internet Mail Access Protocol)交互式邮件存取协议
4. 反向代理
   - 服务的是客户端
5. 正向代理
   - 服务的是服务端

#### 常见服务器对比

1. ISS
   - 互联网信息服务，由微软提供的基于windows系统的互联网基本服务。windows作为服务器在稳定性与其他一些性能上都不如类UNIX操作系统，因此在需要提高性能web的场合下，ISS可能会被“冷落”
2. Tomcat
   - 是一个运行Servlet和JSP的web应用软件，tomcat技术先进。性能稳定且开放源码，因此深受java爱好者的喜爱并得到部门软件开发商的认可，成为目前比较流行的web应用服务器。但是Tomcat天生是一个重量级的web服务器，对静态文件和高并发的处理比价弱
3. Apache
   - 优点：稳定，开源，跨平台
   - 缺点：重量级，不支持高并发，如果请求过多会导致服务器 消耗大量能存，在操作系统的Apache进程之间切换也会消耗大量的CPU资源，并导致HTTP请求的平均响应速度降低
4. Lighttpd
   - 轻量级，高性能，资源没有Nginx丰富 

#### Nginx的功能特性

- 速度更快，并发更高
  - 采用了多进程和I/O多路复用（epoll）的底层实现
- 配置简单，拓展性强
  - 有多模块组成，通过配置文件来添加
  - 还可以开发服务自己业务特性的定制模块
- 高可靠性
  - 多进程模式运行，存在master主进程和N多个worker进程
  - 可以手动设置worker进程数量，每个worker进程之间相互独立提供服务
  - master可以在某个worker进程出错时，可以快读拉起新的worker进程提供服务
- 热部署
  - 可以在Nginx不停止的情况下，对Nginx进行文件升级，更新配置和更换日志文件等功能
- 成本低，BSD许可证
  - 流行的有GPL，BSD，MIT，Mozilla，Apache，LGPL
  - 上面几种的差别
    - 其他人修改源码后，可以闭源：BSD许可证，MIT许可证，Apache许可证
      - 每一个修改过的文件，必须放置版权说明：Apache许可证（BSD许可证不需要，MIT不需要）
        - 衍生软件的广告可以使用你的名字促销：MIT（BSD不可以）
    - 不可以闭源：LGPL，Mozilla，GPL
    - 新增代码可以采用相同的许可证：GPL
      - 需要对源码的修改之处，提供说明文档：Mozilla

#### Nginx的常用功能

- 基本HTTP服务

  - 处理静态文件，处理索引文件以及支持自动索引
  - 提供反向代理服务器，并可以使用缓存加上反向代理，同事完成负载均衡和容错
  - 提供对FastCGI，memcached等服务的缓存机制，同时完成负载均衡和容错
  - 使用Nginx的模块特性特工过滤功能，Nginx基本过滤器包括gzip压缩，ranges支持，chunked响应，XSLT，SSI以及图像缩放等，其中针对包含多个SSI的页面，经由FastCGI或者反向代理，SSI过滤器可以并行处理
  - 支持HTTP下的安全套接层安全协议SSL
  - 支持基于甲醛和依赖的游仙区的HTTP/2

- 高级HTTP服务

  - 支持基于名字和IP的虚拟主机设置
  - 支持HTTP/1.0中的KEEP-Alive模式和管线（PipeLined）模型连接
  - 自定义访问日志格式，带缓存的日志写操作以及快速日志轮转
  - 提供3xx~5xx错误代码重定向功能
  - 支持重写（Rewrite）模块拓展
  - 支持重新加载配置以及在线升级时无需终端正在处理的请求
  - 支持网络监控
  - 支持FLV和MP4流媒体传输

- 邮件服务

  - 支持IMPA/POP3代理服务功能
  - 支持内部SMTP代理服务功能

- :bulb:常用的功能模块

  - ```java
    静态资源部署
    Rewrite地址重写
    	正则表达式
    反向代理
    负载均衡
    	轮询，加权轮询，ip_hash，url_hash，fair
    web缓存
    环境部署
    	高可用环境
    用户认证模块
    ```

- Nginx的核心组成

  ```txt
  nginx二进制可执行文件
  nginx.conf配置文件
  error.log错误的日志记录
  access.log访问日志记录
  ```

### 环境准备

- 前期准备

  - 如果是在linux中需要关闭防火墙设置

    - ```shell
      systemctl stop firewalld 关闭运行的防火墙，系统重新启动之后，防火墙重新打开
      systemctl disable firewalld 永久关闭防火墙，系统重新启动后，防火墙依旧关闭
      ```

  - 确认停用selinux

    - ```shell
      sestatus 查询selinux的状态
      vim /etc/selinux/config selnux的配置文件，设置selinux = disable
      ```

  - GCC编译器

    - 开源的编译期集合，用于处理各种语言，包含了C语言
      - yum install -y gcc

  - PCRE

    - Nginx使用过程中需要使用PCRE库（兼容正则表达式库）
      - yum install -y pcre pcre-devel
      - 使用rpm -qa pcre pcre-devel查看是否安装成功

  - zlib

    - 提供了压缩算法，Nginx的各个模块需要使用gzip压缩，需要提前安装其库及源代码zlib和zlib-devel
    - yum install -y zlib-devel
    - Rpm -qa zlib zlib-devel查看是否安装成功

  - OpenSSl

    - 开放源代码的软件库包可以给程序添加安全通信预防被窃听
    - yum install -y openssl-devel

#### Nginx的例来版本介绍

#### 获取Nginx不同版本的源码

#### 在CentOs上编译安装Nginx

- nginx的简单安装和yum安装差别

  ```shell
  ./nginx -V
  ```

  - 简单安装没有configure argument中的配置文件
    - auto目录
      - 编译相关的文件
    - channges
      - nginx的版本变更记录
    - changges.ru
      - 使用俄罗斯语言写的版本变更记录
    - conf
      - 手动配置配置属性
    - configure
      - 可以检测环境
      - 可以生成相应环境需要的makefile文件

- configure的配置

  - 复杂安装使用到的配置

#### Nginx的源码复杂安装

​	通过./configure来对编译参数进行设置，需要我们手动来指定

- PATH:和路径相关的配置信息

- with:启动模块，默认关闭

- without:关闭模块，默认开启

- --prefix=path

  - ```txt
    指向Nginx的安装目录，默认值为/usr/local/nginx
    ```

- --sbin-path=path

  - ```txt
    指向（执行）程序文件(nginx)的路径，默认值为<prefix>/sbin/nginx
    ```

- --modules-path=path

  - ```txt
    指向Nginx动态模块安装目录，默认值为<prefix>/modules
    ```

- --conf-path= PATH

  - ```txt
    指向配置文件(nginx.conf)的路径。默认值为<prefix>/conf/nginx.conf
    ```

- --error-log-path=PATH

  - ```txt
    指向错误日志文件的路径，默认值<prefix>/logs/error.log
    ```

- --http-log-path=PATH

  - ```txt
    指向访问日志路径，默认值为<prefix>/logs/access.log
    ```

- --pid-path=PATH

  - ```txt
    指向Nginx启动后进行id的文件路径，默认值为<prefix>/logs/nginx.pid
    ```

- --lock-path=PATH

  - ```txt
    指向nginx锁文件的存放路径，默认值为<prefix>/logs/nginx.lock
    ```

- 和path相关的一般用于设置路径
- with相关的是添加模块
  
  - 一般用于添加第三模块设置
- without相关的一般是去除某些模块

#### Nginx重新安装

1. nginx的进程关闭

   ```
   ./nginx -s stop
   ```

2. 将安装的nginx进行删除

   ```shell
   rm -rf /usr/local/nginx
   ```

3. 将安装包之前编译的环境清除

   ```shell
   make clean
   ```

#### Nginx服务器的启停控制

Nginx中的的master和worker进程关系和作用

```java
master主要是管理worker进程
worker负责接收用户的请求，负责处理请求
```

nginx的工作方式

如何获取进程的PID

```shell
ps -ef 
more ../logs/nginx/nginx.pid
```

信号有哪些

| 信号     | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| TERM/INT | 立即关闭整个服务                                             |
| QUIT     | 优雅关闭整个服务                                             |
| HUP      | 重读配置文件并使用服务对新配置项生效                         |
| USR1     | 重新打开日志文件，可以用来进行日志切割                       |
| USR2     | 平滑升级到最新版本的nginx                                    |
| WINCH    | 所有子进程不在接受处理心连接，相当于给woek进行发送了QUIT指令，只关闭worker进程 |

如何通过信号控制nginx的启停等相关操作

```shell
kill  -signal PID
```

Signal:为信号；pid为获取到的master线程id

1. nginx服务的信号控制
   1. 看上面:arrow_double_up:
   2. USR2:d的使用
      1. 会新打开一个master
      2. 将新的pid记录在nginx.pid下，旧的master会存在nginx.pid.oldbin
      3. 在升级成功之后会执行一个QUIT信号，将旧的master优雅的关闭
2. nginx的命令行控制
   1. -?和-h 显示帮助信息
   2. -v 打印版本信号并退出
   3. -V 打印版本信号和配置信息并退出
   4. -t 测试nginx的配置文件语法是否正确并退出
   5. -T 测试nginx的配置文件语法是否正确并列出用到的配置文件信息并退出
   6. -q 在配置测试期间禁止显示非错误信息
   7. -s signal信号，后面可以跟
      1. stop 快速关闭
      2. quit 优雅关闭
      3. reopen 重新打开日志文件类似于USR1
      4. reload 类似于HUP作用
   8. -p  prefix 指定nginx的prefix路径 默认：/usr/local/nginx/
   9. -c filename 指定nginx的配置文件路径 默认 conf/nginx.conf
   10. -g 用来补充nginx配置文件，向nginx服务指定启动时应用全局的配置

### 结构分析

#### 目录结构

1. #### conf：nginx所有配置文件目录

   1. CGI通用网管接口，主要解决的问题是从客户端发送一个请求和数据，服务端获取到请求和数据后调用CGI【程序】处理及相应结果给客户端的一种规范
   2. fastcgi相关配置
      1. fastcgi.conf:fastcgi相关配置文件
      2. fastcgi.conf.default:fastcgi.conf的备份文件
      3. fastcgi_params:fastcgi的参数文件
      4. fastcgi_params.default:fastcgi的参数备份文件
   3. scgi相关配置
      1. scgi_params:scgi的参数文件
      2. scgi_params.default:scgi的参数备份文件
   4. uwsgi相关配置 
      1. uwsgi_params:uwsgi的参数文件
      2. uwsgi_params.default:uwsgi的参数备份文件
   5. koi-utf:编码相关文件
      1. koi-utf
      2. Koi-win
      3. Win-utf
   6. Mine.type:mine类型和固定关系，配置头信息会用到
      1. mine.type
      2. mine.type.default
   7. nginx.conf:nginx最主要的配置文件
      1. nginx.conf
      2. nginx.conf.default

2. #### html目录：成功或失败的页面

3. #### logs目录

   1. access.log 访问日志
   2. error.log 错误日志
   3. nginx.pid nginx进程的PID

4. #### sbin

   1. nginx：启动使用的二进制文件

#### Nginx服务器版本升级和新增模块

需求 将nginx的版本从1.14.2升级到1.16.1，要求nginx不能中断提供服务

环境准备

​	将1.1.4.2安装，将1.16.1编译，不安装

1. 使用nginx服务信号完成nginx的升级

   1. 将1.14.2版本的sbin目录下的nginx进行备份

      ```shell
      cd /usr/local/nginx/sbin
      mv nginx nginxold
      ```

   2. 将ngixn1.16.1安装目录编译后的objs目录下的nginx文件，拷贝到原来 /usr/local/nginx/sbin 目录下

      ```shell
      cd ~/nginx/core/nginx-1.16.1/objs
      cp nginx /usr/local/nginx.sbin
      ```

   3. 发送信号USR2给nginx的1.14.2版本对应的master进程

      ```shell
      kill -USR2 'more /nginx/local/logs/nginx.pid'
      ```

   4. 发送信号QUIT给nginx的1.14.2版本对应的master进程

      ```shell
      kill -QUIT 'more /usr/local/logs/nginx.pid.oldbin'
      ```

2. 使用nginx安装目录的make命令完成升级

   1. 前两步一样
   2. 进入到安装目录，执行make upgrade
   3. 查看是否更新成功

https://www.bilibili.com/video/BV1ov41187bq/?p=21&spm_id_from=pageDriver&vd_source=15cac809b169713f965c1032f507b775

#### 配置文件结构

### 基础配置

#### nginx.conf详解

#### Nginx服务器基础配置案例

## Nginx进阶

### Nginx静态资源部署

#### 静态资源概述

#### 静态资源配置语法

#### 静态资源压缩实战

#### 浏览器缓存

#### 静态资源跨域

#### 静态资源防盗链

### Nginx后端服务器组的配置指令

#### 5个常用指令

#### Rewrite功能配置

##### "地址重写"与"地址转发"

##### Rewrite规则

##### if指令

##### break指令

##### rewrite指令

##### rewrite_log指令

##### set指令

##### Rewrite常用全局变量

#### Rewrite的使用

##### 域名跳转

##### 域名镜像

##### 独立域名

##### 目录自动添加"/"

##### 目录合并

##### 防盗链

### Nginx反向代理

#### 反向代理概述

#### 反向代理配置语法

#### 反向代理实战

#### upstream+proxy_pass

#### 安全隔离

#### 基于原始IP地址阻止流量及并发数

### Nginx负载均衡

#### 负载均衡的概述

#### 负载均衡的原理及处理流程

#### 负载均衡状态

#### 负载均衡的策略

#### 负载均衡的四层流程与原理分析

#### 配置实例一：对所有的请求实现一般轮询规则的负载均衡

#### 配置实例二：对所有的请求实现加权轮询规则的负载均衡

#### 配置实例三：对特定资源实现负载均衡

#### 配置实例四：对不同域名实现负载均衡

#### 配置实例五：实现带有URL重写的负载均衡

### Nginx缓存集成

#### Nginx缓存概述

#### Nginx缓存设置语法

#### Nginx缓存案例

#### Nginx清除缓存

#### Nginx设置页面不缓存

## Nginx架构

### Nginx集群

#### 通过Nginx实现服务器集群搭建

### 高可用解决方案

#### 方案介绍

#### keepalived高可用服务介绍

#### keepalived原理分析

#### VRRP原理解析

#### 高可用环境搭建

##### 环境准备

##### keepalived高可用配置文件详解

##### 效果解析

### Nginx制作下载站点模块

### Nginx用户认证模块

## Nginx模块

### Nginx-Lua模块

#### Lua

##### Lua概述

##### Lua语法

##### Lua数据类型

##### Lua表达式

##### Lua变量

##### Lua流量控制

##### Lua函数

##### Lua其他操作

#### Lua-Nginx-Module概述

#### 安装

#### ngx-lua指令分析

#### ngx-lua的应用场景

#### 案例