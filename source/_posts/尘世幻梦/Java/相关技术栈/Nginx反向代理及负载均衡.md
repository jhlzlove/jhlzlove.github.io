---
title: Nginx
categories:
  - Java
  - Nginx
abbrlink: f7ba05e4
---



<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [1 反向代理](#1-反向代理)
	- [1.1 什么是反向代理](#11-什么是反向代理)
	- [1.2 配置反向代理-准备工作](#12-配置反向代理-准备工作)
	- [1.3 配置反向代理](#13-配置反向代理)
- [2 负载均衡](#2-负载均衡)
	- [2.1 什么是负载均衡](#21-什么是负载均衡)
	- [2.2 配置负载均衡-准备工作](#22-配置负载均衡-准备工作)
	- [2.3 配置负载均衡](#23-配置负载均衡)
- [3. 映射资源文件目录](#3-映射资源文件目录)
- [命令](#命令)

<!-- /code_chunk_output -->

## 1 反向代理

### 1.1 什么是反向代理

反向代理（Reverse Proxy）方式是指以[代理服务器](http://baike.baidu.com/item/代理服务器)来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

### 1.2 配置反向代理-准备工作

（1） 启动spring-boot 项目 端口号8080

（2） 访问 localhost:8080 /hello 可以看到消息

### 1.3 配置反向代理

（1）在Nginx主机修改 Nginx 配置文件

```cfg{.line-numbers}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
			listen       80;
			server_name  localhost;
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
				root   html;
			}
			location / {
				proxy_pass   http://127.0.0.1:8080;
			}
		}
	}
```

（2）重新启动Nginx 然后用浏览器测试：localhost

## 2 负载均衡

### 2.1 什么是负载均衡

负载均衡 建立在现有网络结构之上，它提供了一种廉价有效透明的方法扩展[网络设备](http://baike.baidu.com/item/网络设备)和[服务器](http://baike.baidu.com/item/服务器)的带宽、增加[吞吐量](http://baike.baidu.com/item/吞吐量)、加强网络数据处理能力、提高网络的灵活性和可用性。

负载均衡，英文名称为Load Balance，其意思就是分摊到多个操作单元上进行执行，例如Web[服务器](http://baike.baidu.com/item/服务器)、[FTP服务器](http://baike.baidu.com/item/FTP服务器)、[企业](http://baike.baidu.com/item/企业)关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。

### 2.2 配置负载均衡-准备工作

（1）将刚才的工程的 修改端口号 8081 .8082 分别 启动(同时启动3个应用)

（1）为了能够区分是访问哪个服务器的网站，可以在返回数据上加上端口号区分

### 2.3 配置负载均衡

修改 Nginx配置文件：

```properties{.line-numbers}
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;
	
	upstream tanhua {
	   server localhost:8080;
	   server localhost:8081 ;
	   server localhost:8082;
    }
    server {
			listen       80;
			server_name  localhost;
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
				root   html;
			}
			
			location / {
				proxy_pass   http://tanhua;
			}
		}
}
```

地址栏输入localhost 刷新观察每个网页的标题，看是否不同。

经过测试，三台服务器出现的概率各为33.3333333%，交替显示。

如果其中一台服务器性能比较好，想让其承担更多的压力，可以设置权重。

比如想让NO.1出现次数是其它服务器的2倍，则修改配置如下：

```properties{.line-numbers}
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;
	
	upstream tanhua {
	   server localhost:8080;
	   server localhost:8081 weight=2;
	   server localhost:8082;
    }
    server {
			listen       80;
			server_name  localhost;
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
				root   html;
			}
			
			location / {
				proxy_pass   http://tanhua;
			}
		}
}
```

## 3. 映射资源文件目录

我们可以在服务器上使用Nginx映射出一个目录作为共享的目录，然后访问该可以进行下载。有许多网站使用了该技术。

```properties{.line-numbers}
server {
        listen       8181;
        server_name  localhost;
		#配置跨域
		#add_header Access-Control-Allow-Origin *;
		#add_header Access-Control-Allow-Headers X-Requested-With;
		#add_header Access-Control-Allow-Methods GET,POST,OPTIONS;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /share {
		# /share 的上级全目录
		root /data;
		# 如果使用 alias，最后一定要加 /，而且只能用于 location 块
		# alias /data/share/
		# 显示索引，nginx内部的简单索引
		autoindex on;
		# 显示文件的时间
		autoindex_localtime on;
        #    root   html;
        #   index  index.html index.htm;
        }
}
```

> 在配置映射目录的时候，容易出现问题。因为在 location 块中可以使用两种目录配置方式。一种是 `root`，一种是 `alias`。
> 如果使用的是 root（**不是root用户**），那么 `location` 后面应该填写 URL 访问的路径，而 root 后跟上级路径。
> 如果使用的 alias，那么后面直接写共享的绝对路径即可，注意在绝对路径后加 ”/“，而 location 后依然只写 URL 访问路径即可。
> 以这里的配置为例。那么我访问的地址应该是 `xxx.com:8181/share`，我的共享目录是 `/root/share`。

## 命令

```bash{.line-numbers}
# 停止
./nginx -s stop

# 重启
./nginx -s reload
```
