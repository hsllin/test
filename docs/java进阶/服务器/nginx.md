
引用自# [Nginx 常用配置汇总！从入门到干活足矣](https://segmentfault.com/a/1190000040054327)

### nginx介绍
`Nginx`是一个高性能的`http`和反向代理服务器，其特点是占用内存小，并发能力强。`Nginx`专为性能优化而开发，性能是其最重要的考量，能经受高负载的考验，有报告表明能支持高达50000个并发连接数。Nginx通常被用来实现**正向代理**,**反向代理，负载均衡，以及动静分离**这四个功能。

#### 正向代理
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230622171142.png)

#### 反向代理
反向代理（Reverse Proxy）方式是指以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230622171150.png)

#### 负载均衡

如果请求数过大，单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器的情况改为请求分发到多个服务器上，就是负载均衡。

#### 动静分离

为了加快服务器的解析速度，可以把动态页面和静态页面交给不同的服务器来解析，加快解析速度，降低原来单个服务器的压力。

小技巧补充：域名匹配的四种写法


```shell
精确匹配：server_name www.mingongge.com ;
左侧通配：server_name *.mingongge.com ;
右侧统配：server_name www.mingongge.* ;
正则匹配：server_name ~^www\.mingongge\.*$ ;
```


匹配优先级：精确匹配 > 左侧通配符匹配 > 右侧通配符匹配 > 正则表达式匹配

#### 静态资源配置


```nginx
server {  
listen 80;  
server_name mingongge.com;  
location /static {      
  root /wwww/web/web_static_site; 
  }
}
```

#### 反向代理

比如生产环境（同一台服务中）有不同的项目，这个就比较实用了，用反向代理去做请示转发。


```nginx
http {
.............
    upstream product_server{
        127.0.0.1:8081;
    }

    upstream admin_server{
        127.0.0.1:8082;
    }

    upstream test_server{
        127.0.0.1:8083;
    }

server {
      
  #默认指向product的server
  location / {
      proxy_pass http://product_server;
      }

  location /product/{
      proxy_pass http://product_server;
     }

  location /admin/ {
      proxy_pass http://admin_server;
     }

  location /test/ {
      proxy_pass http://test_server;
      }
    }
}
```

#### 负载均衡


```nginx
upstream server_pools { 
  server 192.168.1.11:8880   weight=5;
  server 192.168.1.12:9990   weight=1;
  server 192.168.1.13:8989   weight=6;
  #weigth参数表示权值，权值越高被分配到的几率越大
}
server {  
  listen 80; 
  server_name mingongge.com;
  location / {    
  proxy_pass http://server_pools; 
   }
}
```

#### 代理相关的其它配置


```shell
proxy_connect_timeout 90;  #nginx跟后端服务器连接超时时间(代理连接超时)
proxy_send_timeout 90;     #后端服务器数据回传时间(代理发送超时)
proxy_read_timeout 90;     #连接成功后,后端服务器响应时间(代理接收超时)
proxy_buffer_size 4k;      #代理服务器（nginx）保存用户头信息的缓冲区大小
proxy_buffers 4 32k;      #proxy_buffers缓冲区
proxy_busy_buffers_size 64k;     #高负荷下缓冲大小（proxy_buffers*2）
proxy_temp_file_write_size 64k;  #设定缓存文件夹大小

proxy_set_header Host $host; 
proxy_set_header X-Forwarder-For $remote_addr;  #获取客户端真实IP
```

## 高级配置

#### 重定向配置


```nginx
location / { 
   return 404; #直接返回状态码
}
location / { 
   return 404 "pages not found"; 
   #返回状态码 + 一段文本
}
location / { 
   return 302 /blog ; 
   #返回状态码 + 重定向地址}
location / { 
   return https://www.mingongge.com ; 
   #返回重定向地址
  }
```


示例如下


```nginx
server { 
listen 80;
server_name www.mingongge.com;
return 301 http://mingongge.com$request_uri;
}
server {
listen 80; 
server_name www.mingongge.com; 
location /cn-url { 
   return 301 http://mingongge.com.cn; 
   }
}

server{
  listen 80;
  server_name mingongge.com; # 要在本地hosts文件进行配置
  root html;
  location /search {
   rewrite ^/(.*) https://www.mingongge.com redirect;
  }
  
  location /images {
   rewrite /images/(.*) /pics/$1;
  }
  
  location /pics {
   rewrite /pics/(.*) /photos/$1;
  }
  
  location /photos {
  
  }
}
```


#### 设置缓冲区容量上限

这样的设置可以阻止缓冲区溢出攻击（同样是Server模块）


```nginx
client_body_buffer_size 1k;
client_header_buffer_size 1k;
client_max_body_size 1k;
large_client_header_buffers 2 1k;

#设置后，不管多少HTTP请求都不会使服务器系统的缓冲区溢出了
```


#### 限制最大连接数

在http模块内server模块外配置limit_conn_zone，配置连接的IP,在http，server或location模块配置limit_conn，能配置IP的最大连接数。


```nginx
limit_conn_zone $binary_remote_addr zone=addr:5m;
limit_conn addr 1;
```

#### SSL 证书配及跳转HTTPS配置


```nginx
server {
      listen 192.168.1.250:443 ssl;
      server_tokens off;
      server_name mingonggex.com www.mingonggex.com;
      root /var/www/mingonggex.com/public_html;
      ssl_certificate /etc/nginx/sites-enabled/certs/mingongge.crt;
      ssl_certificate_key /etc/nginx/sites-enabled/certs/mingongge.key;
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
      
}
# Permanent Redirect for HTTP to HTTPS
server 
{  
listen 80;  
server_name mingongge.com; 
return 301  https://$server_name$request_uri;
}
```


## 安全配置

#### 禁用server_tokens项

server_tokens在打开的情况下会使404页面显示Nginx的当前版本号。这样做显然不安全，因为黑客会利用此信息尝试相应Nginx版本的漏洞。只需要在nginx.conf中http模块设置server_tokens off即可，例如：

```nginx
server {
    listen 192.168.1.250:80;
    Server_tokens off;
    server_name mingongge.com www.mingongge.com;
    access_log /var/www/logs/mingongge.access.log;
    error_log /var/www/logs/mingonggex.error.log error;
    root /var/www/mingongge.com/public_html;
    index index.html index.htm;
}
#重启Nginx后生效：
```


#### 禁止非法的HTTP User Agents

User Agent是HTTP协议中对浏览器的一种标识，禁止非法的User Agent可以阻止爬虫和扫描器的一些请求，防止这些请求大量消耗Nginx服务器资源。

为了更好的维护，最好创建一个文件，包含不期望的user agent列表例如/etc/nginx/blockuseragents.rules包含如下内容：

```nginx

map $http_user_agent $blockedagent {
   default 0;
    ~*malicious 1;
    ~*bot 1;
    ~*backdoor 1;
    ~*crawler 1;
    ~*bandit 1;
}

```

然后将如下语句放入配置文件的server模块内


```nginx
include /etc/nginx/blockuseragents.rules;

```

并加入if语句设置阻止后进入的页面：

#### 阻止图片外链


```nginx
location /img/ {
      valid_referers none blocked 192.168.1.250;
        if ($invalid_referer) {
          return 403;
      }
}
```


#### 封杀恶意访问

[挺带劲！通过 Nginx 来实现封杀恶意访问](https://link.segmentfault.com/?enc=jM%2B%2B5xxUoRo7jcWhOGHHOQ%3D%3D.VF%2BTenBL25A%2BMU2Y%2FQPvMNNJJloGKfKwA9fKKEpKYJy8Cd9b%2F9pe7XCCgazIPkBY3gkQmJbk5kz%2BubXCSluaJ4RbDshjk94B5o5yALfSHTHW4nXKQJfFWQ9eh3MayhTkaIPpwfyv6uYc%2FjGfQVgD%2BojVLFm%2FvClKQ1IKn9z7n9kpkRxhVqzMxWoFRxJK4XoeMUkZIDllm5FVcn4QwLv7ygb25P6m7nE36iXQ7%2FByNesjbHLFh64jZq3MdQ1RaHqj4OV7f5toijzj6TpwUSr5I7cviawkS57LYJNwD18gt9nmZQB4kP7wTm5OnHXEQ1A9nkMSrK2UBi3JBwz6FkIt6w%3D%3D)

#### 禁掉不需要的 HTTP 方法

一些web站点和应用，可以只支持GET、POST和HEAD方法。在配置文件中的 serve r模块加入如下方法可以阻止一些欺骗攻击


```nginx
if ($request_method !~ ^(GET|HEAD|POST)$) {
    return 444;
}
```


#### 禁止 SSL 并且只打开 TLS

尽量避免使用SSL，要用TLS替代，以下配置可以放在Server模块内


```nginx
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
```
