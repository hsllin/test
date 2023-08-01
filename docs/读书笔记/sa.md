[![](https://7-shu.com/wp-content/uploads/2022/12/950x185_banner.webp)](https://www.namesilo.com/?rid=5794483pd)

很多时候我们使用了便宜的主机，但这种主机相应速度实在感人，这里就要用到中转来恢复我们主机的速度。不过也有朋友会觉着这个方法不划算太折腾，还不如直接买个CN2 GIA。确实！我也觉得哈哈?

还是言归正传，相信还是有很多小伙伴有这个需求，这里我们就来讲解一下具体操作！首先这里会用到两台主机：  
1.垃圾vps本身（已经部署v2ray或trojan）  
2.中转主机（这里用到的机器可以是NAT, VPS , VDS只要你觉得速度不错，配置不用太高，最好是国内主机连接速度快就好）

___

在你的NAT机上安装Iptables，这里我们使用一键脚本就好，安装速度还是蛮快！执行以下脚本

```
wget -qO natcfg.sh http://arloor.com/sh/iptablesUtils/natcfg.sh && bash natcfg.sh
```

或者

```
wget -qO natcfg.sh https://raw.githubusercontent.com/arloor/iptablesUtils/master/natcfg.sh && bash natcfg.sh
```

脚本执行以后会出现以下画面

```
连接成功
[root@vm619974 ~]# wget -qO natcfg.sh http://arloor.com/sh/iptablesUtils/natcfg.sh && bash natcfg.sh
用途: 便捷的设置iptables端口转发
注意1: 到域名的转发规则在添加后需要等待2分钟才会生效，且在机器重启后仍然有效
注意2: 到IP的转发规则在重启后会失效，这是iptables的特性

你要做什么呢（请输入数字）？Ctrl+C 退出本脚本
1) 增加到域名的转发      3) 增加到IP的转发        5) 列出所有到域名的转发
2) 删除到域名的转发      4) 删除到IP的转发        6) 查看iptables转发规则
```

这里可以转发域名或IP，并可以转发多个，我们这里以域名为例！选择输入1 增加到域名的转发

```
#? 1
本地端口号:35223    // 这里的端口号是这台中转机器上所需要使用的端口号。注意防火墙请放行这个端口号，请更改成你所使用的端口号！
远程端口号:41000    // 这里的端口号输入垃圾VPS上v2ray或trojan所使用的端口号，请改成你的！
目标域名:name.com   // 这里是你解析到垃圾VPS上所使用的域名，请输入你的域名
```

输入完以后回车会有如下提示：

```
成功添加转发规则 35223>name.com:41000 大约两分钟后规则会生效
```

这样转发就已经成功！回到你v2ray或trojan客户端，改下里面的信息把域名改成这台中转机的IP就行，其他不用变！若要转发ip操作一样只是把域名换ip，并可添加多个转发！  

[![](https://7-shu.com/wp-content/uploads/2023/02/ad_950x185_banner.webp)](https://hperformence.top/#/register?code=nvparrVv)