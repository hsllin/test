### 开关机

```shell
reboot 重启
```


```shell
shutdown -h 关机 
```
 
```shell
shutdown -h now 马上关机
```

### [#](#查看系统所有用户) 查看系统所有用户

```shell
[root@pdai.tech ~]# cut -d: -f1 /etc/passwd
root
bin
daemon
adm
...
```
管理用户

| 命令       | 功能描述       |
| ---------- | -------------- |
| useradd XX | 创建用户       |
| passwd xx  | 为用户设置密码 |


### 解压缩
**`压缩`**  

```shell
tar –cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg  
tar –czf jpg.tar.gz *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz  
tar –cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2  
tar –cZf jpg.tar.Z *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z  
rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for Linux  
zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux
```


**`解压`**  

```shell
tar –xvf file.tar //解压 tar包  
tar -xzvf file.tar.gz //解压tar.gz  
tar -xjvf file.tar.bz2 //解压 tar.bz2  
tar –xZvf file.tar.Z //解压tar.Z  
unrar e file.rar //解压rar  
unzip file.zip //解压zip
```

 
##### 文件操作

| 命令            |                                 功能描述| 居中对齐 |
|:------------------ | --------------------------------------:|:--------:|
| touch FileName     |                 创建名为FileName的文件 |  单元格  |
| echo 内容>fileName | 把内容覆盖到filename上，若不存在则新建 |  单元格  |
| head -n fileName   |                          查看前n行内容 |          |
| tail -n fileName   |                          查看后n行内容 |          |
| wc fileName        |                           查看文件行数 |          |


#### 查找操作

| 命令            |                                 功能描述| 居中对齐 |
|:------------------ | --------------------------------------:|:--------:|
| find / -name passwd[完整名称]    |                查找passwd文件 |  单元格  |
| find ./ name "p*" | 查找带p的文件 |  单元格  |
| head -n fileName   |                          查看前n行内容 |          |
| tail -n fileName   |                          查看后n行内容 |          |
| wc fileName        |                           查看文件行数 |          |


#### 文件监听 - tail

最常用的`tail -f filename`

```shell
# 基本使用
tail -f xxx.log # 循环监听文件
tail -300f xxx.log #倒数300行并追踪文件
tail +20 xxx.log #从第 20 行至文件末尾显示文件内容

# tailf使用
tailf xxx.log #等同于tail -f -n 10 打印最后10行，然后追踪文件
```

tail的参数

```shell
-f 循环读取
-q 不显示处理信息
-v 显示详细的处理信息
-c<数目> 显示的字节数
-n<行数> 显示文件的尾部 n 行内容
--pid=PID 与-f合用,表示在进程ID,PID死掉之后结束
-q, --quiet, --silent 从不输出给出文件名的首部
-s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒
```

### 网络相关

```shell
ifconfig 查看网络地址
```

编译生效  

```shell
  source /etc/profile
```


### [#](#查看所有网络接口的属性) 查看所有网络接口的属性

```shell
[root@pdai.tech ~]# ifconfig
```
#### 防火墙相关
 查看防火墙设置

```shell
[root@pdai.tech ~]# iptables -L

```
centos7:
查看防火墙开启端口

```shell
firewall-cmd --list-ports 
```
 
添加放行端口

```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent 
```
 
重启firewall  

```shell
firewall-cmd --reload  
```


停止firewall  

```shell
systemctl stop firewalld.service  
```

禁止firewall开机启动  

```shell
systemctl disable firewalld.service  
```
  
centos6
查看打开的端口：

```shell
# /etc/init.d/iptables status
```

放行80端口
```shell
# /sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```
保存
```shell
# /etc/init.d/iptables save
```
关闭防火墙
```shell
#/etc/init.d/iptables stop
```

#### 查看路由表

```shell
[root@pdai.tech ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.175.253  0.0.0.0         UG    0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.31.160.0    0.0.0.0         255.255.240.0   U     0      0        0 eth0
```

### [#](#netstat) netstat

查看所有监听端口

```shell
[root@hecs-84361 ~]# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1637/sshd           
tcp        0      0 0.0.0.0:8090            0.0.0.0:*               LISTEN      554291/docker-proxy 
tcp        0      0 0.0.0.0:5984            0.0.0.0:*               LISTEN      275038/docker-proxy 
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      526831/docker-proxy 
tcp6       0      0 :::80                   :::*                    LISTEN      660937/httpd        
tcp6       0      0 :::22                   :::*                    LISTEN      1637/sshd           
tcp6       0      0 :::8090                 :::*                    LISTEN      554297/docker-proxy 
tcp6       0      0 :::5984                 :::*                    LISTEN      275043/docker-proxy 
tcp6       0      0 :::3306                 :::*                    LISTEN      526837/docker-proxy 
       
```

查看所有已经建立的连接

```
[root@hecs-84361 ~]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1637/sshd           
tcp        0      0 0.0.0.0:8090            0.0.0.0:*               LISTEN      554291/docker-proxy 
tcp        0      0 0.0.0.0:5984            0.0.0.0:*               LISTEN      275038/docker-proxy 
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      526831/docker-proxy 
tcp        0    264 192.168.0.69:22         113.65.228.38:20775     ESTABLISHED 2088964/sshd: root  
tcp6       0      0 :::80                   :::*                    LISTEN      660937/httpd        
tcp6       0      0 :::22                   :::*                    LISTEN      1637/sshd           
tcp6       0      0 :::8090                 :::*                    LISTEN      554297/docker-proxy 
tcp6       0      0 :::5984                 :::*                    LISTEN      275043/docker-proxy 
tcp6       0      0 :::3306                 :::*                    LISTEN      526837/docker-proxy 
```

查看当前连接

```shell
[root@pdai.tech ~]# netstat -nat|awk  '{print $6}'|sort|uniq -c|sort -rn
      5 LISTEN
      2 ESTABLISHED
      1 Foreign
      1 established)
```

查看网络统计信息进程

```
[root@pdai.tech ~]# netstat -s
Ip:
    Forwarding: 1
    19549848 total packets received
    2 with invalid addresses
    18056701 forwarded
    0 incoming packets discarded
    1493084 incoming packets delivered
    19319964 requests sent out
    139 dropped because of missing route

...
```

netstat 请参考这篇文章: [Linux netstat命令详解在新窗口打开](https://www.cnblogs.com/ftl1012/p/netstat.html)

### [#](#查看所有进程) 查看所有进程

```shell
[root@pdai.tech ~]# ps -ef | grep java
root      1249     1  0 Nov04 ?        00:58:05 java -jar /opt/tech_doc/bin/tech_arch-0.0.1-RELEASE.jar --server.port=9999
root     32718 32518  0 08:36 pts/0    00:00:00 grep --color=auto java
```

### [#](#top) top

top除了看一些基本信息之外，剩下的就是配合来查询vm的各种问题了

```shell
# top -H -p pid
top - 08:37:51 up 45 days, 18:45,  1 user,  load average: 0.01, 0.03, 0.05
Threads:  28 total,   0 running,  28 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882088 total,    74608 free,   202228 used,  1605252 buff/cache
KiB Swap:  2097148 total,  1835392 free,   261756 used.  1502036 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 1347 root      20   0 2553808 113752   1024 S  0.3  6.0  48:46.74 VM Periodic Tas
 1249 root      20   0 2553808 113752   1024 S  0.0  6.0   0:00.00 java
 1289 root      20   0 2553808 113752   1024 S  0.0  6.0   0:03.74 java
...
```

### [#](#查看磁盘和内存相关) 查看磁盘和内存相关

#### [#](#查看内存使用-free-m) 查看内存使用 - free -m

```shell
[root@pdai.tech ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1837         196         824           0         816        1469
Swap:          2047         255        1792
```

### [#](#查看各分区使用情况) 查看各分区使用情况

```shell
[root@pdai.tech ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        909M     0  909M   0% /dev
tmpfs           919M     0  919M   0% /dev/shm
tmpfs           919M  452K  919M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda1        40G   15G   23G  40% /
tmpfs           184M     0  184M   0% /run/user/0
```

### [#](#查看指定目录的大小) 查看指定目录的大小

```
[root@pdai.tech ~]# du -sh
803M
```

### [#](#查看内存总量) 查看内存总量

```shell
[root@pdai.tech ~]# grep MemTotal /proc/meminfo
MemTotal:        1882088 kB
```

### [#](#查看空闲内存量) 查看空闲内存量

```shell
[root@pdai.tech ~]# grep MemFree /proc/meminfo
MemFree:           74120 kB
```

### [#](#查看系统负载磁盘和分区) 查看系统负载磁盘和分区

```shell
[root@pdai.tech ~]# grep MemFree /proc/meminfo
MemFree:           74120 kB
```

### [#](#查看系统负载磁盘和分区-1) 查看系统负载磁盘和分区

```shell
[root@pdai.tech ~]# cat /proc/loadavg
0.01 0.04 0.05 2/174 32751
```

### [#](#查看挂接的分区状态) 查看挂接的分区状态

```shell
[root@pdai.tech ~]# mount | column -t
sysfs       on  /sys                             type  sysfs       (rw,nosuid,nodev,noexec,relatime)
proc        on  /proc                            type  proc        (rw,nosuid,nodev,noexec,relatime)
devtmpfs    on  /dev                             type  devtmpfs    (rw,nosuid,size=930732k,nr_inodes=232683,mode=755)
securityfs  on  /sys/kernel/security             type  securityfs  (rw,nosuid,nodev,noexec,relatime)
...
```

### [#](#查看所有分区) 查看所有分区

```shell
[root@pdai.tech ~]# fdisk -l

Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0008d73a

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83884031    41940992   83  Linux
```

### [#](#查看所有交换分区) 查看所有交换分区

```shell
[root@pdai.tech ~]# swapon -s
Filename                                Type            Size    Used    Priority
/etc/swap                               file    2097148 261756  -2
```

### [#](#查看硬盘大小) 查看硬盘大小

```shell
[root@pdai.tech ~]# fdisk -l |grep Disk
Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Disk label type: dos
Disk identifier: 0x0008d73a
```


