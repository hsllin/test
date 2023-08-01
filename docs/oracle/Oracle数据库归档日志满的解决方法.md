早上收到学校现场同同事发来的消息，说考勤页面打开报错 ，重启过好几次不生效。

<img src="https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/lQDPJyJWHVOBzQbNCSTNBDiw-2FknAbcVDcEf3hU2gDGAA_1080_2340.jpg" width="240px" />


经过一顿分析排查，发现考勤服务是正常的，在请求权限平台获取用户的时候报错，然后就去看权限平台，发现登录报错了。
**错误详情**

```java
ORA-00257: archiver error. Connect internal only, until freed
```


然后网上搜索这个错误，发现原来是Archiver日志满了，在部署了oracle服务的机器上查看,发现果然archivelog已经占用100%，满了。

```shell
# dh -h

```

![1686706149445_329D363F-A0FE-48d9-B3A6-F2F40461C61C.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/1686706149445_329D363F-A0FE-48d9-B3A6-F2F40461C61C.png)


以下是一些查询相关日志的操作命令
### 辅助工作 / sqlplus

### 查看 archive log
show parameter log_archive_dest;

### 查看 log sequence
archive log list;

### 查看 flash recovery area
select * from V$FLASH_RECOVERY_AREA_USAGE;

### 查看日志所在目录及日志空间设置
show parameter db_recovery_file_dest;

清理日志 / rman

rman target sys/password

### 检查无用的日志
crosscheck archivelog all;

### 删除过期的日志
delete expired archivelog all;

### 删除15天前的日志
delete archivelog until time 'sysdate-15' ;

### 删除 log sequence 为15及15之前的所有归档日志
delete archivelog until sequence 15;

### 删除15天以前的归档日志，但不删除闪回区有效的归档日志
delete archivelog all completed before 'sysdate-15';

### 清除所有归档日志
delete noprompt archivelog all completed before 'sysdate';

### 删除指定到某时间的归档日志
delete archivelog until time "to_date('2022-10-09 13:00:00','yyyy-mm-dd hh24:mi:ss')";
