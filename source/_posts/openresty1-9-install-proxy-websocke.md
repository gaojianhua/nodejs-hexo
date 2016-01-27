title: openresty1.9-install 支持反向代理 websocke
date: 2015-10-27 16:42:22
tags:
- 开始
- 我
- 日记
categories: openresty
---


### 环境: 
	centos 6.5

### 下载 
	http://openresty.org/cn/ 官网 下载 最新版本

### 安装依赖
```bash
yum install readline-devel pcre-devel openssl-devel gcc
```

### 执行安装

```bash
tar xzvf ngx_openresty-VERSION.tar.gz
cd ngx_openresty-VERSION/
./configure --prefix=/opt/local/openresty 
make
make install
```
### 启动

```bash
 cd /opt/local/openresty/nginx 
 ./sbin/nginx  -t  
 ./sbin/nginx  
 ```

### 配置nginx.conf

WebSocket 和HTTP协议不同，但是WebSocket中的握手和HTTP中的握手兼容，它使用HTTP中的Upgrade协议头将连接从HTTP升级到WebSocket。这使得WebSocket程序可以更容易的使用现已存在的基础设施。例如，WebSocket可以使用标准的HTTP端口 80 和 443，因此，现存的防火墙规则也同样适用。


一个WebSockets的应用程序会在客户端和服务端保持一个长时间工作的连接。用来将连接从HTTP升级到WebSocket的HTTP升级机制使用HTTP的Upgrade和Connection协议头。反向代理服务器在支持WebSocket方面面临着一些挑战。一项挑战是WebSocket是一个hop-by-hop协议，所以，当代理服务器拦截到一个客户端发来的Upgrade请求时，它(指服务器)需要将它自己的Upgrade请求发送给后端服务器，也包括合适的请求头。此外，由于WebSocket连接是长时间保持的，所以代理服务器需要允许这些连接处于打开状态，而不是像对待HTTP使用的短连接那样将其关闭。
NGINX 通过在客户端和后端服务器之间建立起一条隧道来支持WebSocket。为了使NGINX可以将来自客户端的Upgrade请求发送给后端服务器，Upgrade和Connection的头信息必须被显式的设置。如下所示：


```bash
worker_processes 3;
error_log logs/error.log debug;
events {
        worker_connections 1024;
}
http {

        include mime.types;
        default_type text/html;
        sendfile on;
        keepalive_timeout 65;
        lua_shared_dict config 2m;
        server {
                listen 80;
                server_name localhost;
                root html;
                index index.html index.htm;
                large_client_header_buffers 4 16k;
                location /{
                        root /opt/apps/dist;
                        access_log off;
                        expires off;
                }

                location /push{
                        proxy_pass  http://192.168.0.61/pushtest;

                        #proxy_set_header X-Real-IP $remote_addr;
                        #proxy_set_header Host $host;
                        #proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

                        proxy_http_version 1.1;
                        proxy_set_header Upgrade $http_upgrade;
                        proxy_set_header Connection "upgrade";
                }

        }




}
 ```


### 重启

```bash
 ./sbin/nginx  -s reload 
 ```


参考文档:
	[中文官网](http://openresty.org/cn/) 
	[反向代理websocke介绍](http://www.oschina.net/translate/websocket-nginx) 
	[websocke和redis推送](http://zxh.sx.cn/2015/11/24/technology/2015-11-24-websocket-redis-openresty/) 
