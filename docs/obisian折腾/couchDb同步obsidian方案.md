Obsidian 好用，但是同步太烂了。第三方插件 obsidian-livesync 提供了 CouchDB 同步的功能，这肯定搞一下

___
[连接](https://www.joyk.com/dig/detail/1649775726883308)

## 1、Obsidian

一款 Markdown 编辑器，插件很多,笔记都存在本地，集成了相当多的插件，还有精美的主题，折腾一下可以打造成笔记流的生产利器。


___

## 2、obsidian-livesync

Obsidian-LiveSync 是一款第三方同步插件 \[ [链接](https://github.com/vrtmrz/obsidian-livesync) \]

支持 IBM Cloudant 和 Apache CouchDB ，由于没钱所以我们当然是会选择 CouchDB

首先我们需要在 库 中安装该插件，注意每个库都要这样安装

安装好后我们记得要启用这个插件。

___

## 3、CouchDB

首先当然是安装 CouchDB
采用docker的安装方式

3.1、Docker 版 CouchDB


``` shell
mkdir /opt/couchdb
cd /opt/couchdb
vim local.ini
```


```shell
[couchdb]
single_node=true

[chttpd]
require_valid_user = true

[chttpd_auth]
require_valid_user = true
authentication_redirect = /_utils/session.html

[httpd]
WWW-Authenticate = Basic realm="couchdb"
enable_cors = true

[cors]
origins = app://obsidian.md,capacitor://localhost,http://localhost
credentials = true
headers = accept, authorization, content-type, origin, referer
methods = GET, PUT, POST, HEAD, DELETE
max_age = 3600
```

```shell
docker run --rm -it -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=password -v .local.ini:/opt/couchdb/etc/local.ini -p 5984:5984 couchdb
```


```java
docker run -d --rm -it -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=admin -v /opt/couchdb/etc/obsidian.ini:/opt/couchdb/etc/local.ini -p 5984:5984 couchdb
```

注意如果你忘记创建 local.ini 并直接运行 docker 命令，docker 会因为 -v 参数的挂载而自动生成一个 local.ini 的文件夹，如果想配置还需要删掉才能使用。
```

由于我们创建的只是给 Obsidian 用，所以我们只需要新增一个配置文件并填入相关配置即可

配置参数说明 \[ [链接](https://docs.couchdb.org/en/latest/config-ref.html) \]

Obsidian 用到的配置 \[ [链接](https://github.com/vrtmrz/obsidian-livesync/blob/main/docs/setup_own_server.md) \]

```

配置好后端并重启后，我们在前端配置上数据库即可（需登录）

```
http://192.168.1.10:5984/_utils/
```

```
Setup - Configure a Single Node 配置单用户模式
Create - Create Database 创建数据库
```

创建完数据库，就可以在 Obsidian 配置好参数并使用了

```
URL  http://192.168.1.10:5480
Username  admin
Password  yourpass
Database name  demo
```

也可以创建子用户，仅管理指定数据库，这样分开权限管控。
windows obsidian配置
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619175854.png)


___

## 4、CouchDB 与 Nginx SSL

既然都有 http 了，都 2022 年了，为什么不来一个 https 呢

当然，域名和 Nginx 服务相关的配置就不提了，直接上关键配置 \[ [链接](https://docs.couchdb.org/en/3.2.0/best-practices/reverse-proxies.html) \]

```
原始数据库访问 URL：http://localhost:5984/obsidian/
对外数据库访问 URL：https://your.domain.name/couchdb/obsidian/
```

```
原始访问 URL：http://localhost:5984/
对外访问 URL：https://your.domain.name/couchdb/
应用访问 URL：https://your.domain.name/couchdb
应用数据库 URL：https://your.domain.name/couchdb/obsidian/
```

```
server {
    listen   443 ssl http2;
    listen   [::]:443 ssl http2;
    server_name  your.domain.name;
    root         /usr/share/nginx/html;
    ssl_certificate PATH_TO_YOUR_PUBLIC_KEY.pem;
    ssl_certificate_key PATH_TO_YOUR_PRIVATE_KEY.key;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305';
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout  1d;
    access_log  /var/log/nginx/couchdb.log main buffer=64k flush=15s;
    error_log  /var/log/nginx/couchdb-error.log;

    location / {
    }

    location /couchdb {
        rewrite /couchdb/(.*) /$1 break;
        proxy_pass http://localhost:5984;
        proxy_redirect off;
        proxy_buffering off;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Ssl on;
    }

    location /_session {
        proxy_pass http://localhost:5984/_session;
        proxy_redirect off;
        proxy_buffering off;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Ssl on;
    }
```

主要内容有两部分  
1、后端转发传递给 /couchdb  
2、会话转发传递给 /\_session

___

## 5、特殊问题

配置是配置好了，但是不好用怎么办

首先就得看日志，正常运行时的日志如下所示

[![obsidian_showlog.png](https://blog.starryvoid.com/wp-content/uploads/2022/04/obsidian_showlog.png)](https://blog.starryvoid.com/wp-content/uploads/2022/04/obsidian_showlog.png)

插件内的 Test Database Connection 是发送一个 HTTP 请求，只要有 200 OK 就算连接成功 Connected to test 。

```
Error:Request Error:404 could not connect to https://your.domain.name/couchdb : demo (Error:Request Error:404)
```

所以以日志为准。当日志出现 Replication closed 时基本上可以认为连接失败。

## 5.1、iCloud 同步 Obsidian

在ipad pro上装个obsidian，然后装个obsidian-livesync插件，简直不要太香。
