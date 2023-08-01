[halo官网](https://halo.run/)

安装docker


安装docker-compose

```javascript
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

```

```python
sudo chmod +x /usr/local/bin/docker-compose

```

在系统任意位置创建一个文件夹，此文档以 `~/halo` 为例。

```
mkdir ~/halo && cd ~/halo
```

创建 `docker-compose.yaml`
halo+mysql的配置

```yaml
version: "3"  
  
services:  
halo:  
image: halohub/halo:2.6  
container_name: halo  
restart: on-failure:3  
depends_on:  
halodb:  
condition: service_healthy  
networks:  
halo_network:  
volumes:  
- ./:/root/.halo2  
ports:  
- "8090:8090"  
healthcheck:  
test: ["CMD", "curl", "-f", "http://localhost:8090/actuator/health/readiness"]  
interval: 30s  
timeout: 5s  
retries: 5  
start_period: 30s  
command:  
- --spring.r2dbc.url=r2dbc:pool:mysql://halodb:3306/halo  
- --spring.r2dbc.username=root  
# MySQL 的密码，请保证与下方 MYSQL_ROOT_PASSWORD 的变量值一致。  
- --spring.r2dbc.password=root  
- --spring.sql.init.platform=mysql  
# 外部访问地址，请根据实际需要修改  
- --halo.external-url=http://localhost:8090/  
# 初始化的超级管理员用户名  
- --halo.security.initializer.superadminusername=admin  
# 初始化的超级管理员密码  
- --halo.security.initializer.superadminpassword=hsl6316436  
  
halodb:  
image: mysql:8.0.31  
container_name: halodb  
restart: on-failure:3  
networks:  
halo_network:  
command:  
- --default-authentication-plugin=mysql_native_password  
- --character-set-server=utf8mb4  
- --collation-server=utf8mb4_general_ci  
- --explicit_defaults_for_timestamp=true  
volumes:  
- ./mysql:/var/lib/mysql  
- ./mysqlBackup:/data/mysqlBackup  
ports:  
- "3306:3306"  
healthcheck:  
test: ["CMD", "mysqladmin", "ping", "-h", "127.0.0.1", "--silent"]  
interval: 3s  
retries: 5  
start_period: 30s  
environment:  
# 请修改此密码，并对应修改上方 Halo 服务的 SPRING_R2DBC_PASSWORD 变量值  
- MYSQL_ROOT_PASSWORD=root  
- MYSQL_DATABASE=halo  
  
networks:  
halo_network:

```

