
### 一、问题现象
今天早上测试妹子反应考勤的人脸权限添加报错了，然后跑去看了下日志

```shell
Error 1205: Lock wait timeout exceeded; try restarting transaction
```
然后网上一顿搜，大概就是数据表被锁住了。


### 二、原因分析
这个考勤的权限之前测试一直没问题的，里面的逻辑比较简单，就是将前端传过来的人员部门和工号等数据整理以下，分别存进权限表和权限人员表，涉及到update操作。为啥update操作会导致数据库锁表呢，貌似没说法啊。
一般死锁原因：
1.并发+两个事务+操作同一条数据
2.表索引设计不当，导致数据库出现死锁
3.长事务，阻塞DDL，继而阻塞所有同表的后续操作。
4.执行DML操作没有commit，再执行删除操作就会锁表。

### 三、尝试方法
（之前有结果的图忘记截下来保存了，现在看到的可能没数据了）

#### 3.1查看表使用
```mysql
show open tables where in_use > 0 ;
```
如果查询结果为空，那么说明表没在使用，说明不是锁表的问题
#### 3.2查看当前运行的所有事务
```mysql
SELECT * FROM information_schema.INNODB_TRX;
```
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619161026.png)

#### 3.3查询锁等待的对应关系
```mysql
SELECT * FROM information_schema.INNODB_LOCK_waits;
```
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619161104.png)

#### 3.4查看当前出现的锁
```mysql
SELECT * FROM information_schema.INNODB_LOCKs;
```
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619161120.png)


```mysql
show full PROCESSLIST
```

![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619160818.png)

#### 3.5kill掉事务
kill上面查出来的id

#### 3.6批量 kill processlist
通过information_schema.processlist表中的连接信息生成需要处理掉的MySQL连接的语句临时文件，然后执行临时文件中生成的指令
然后kill掉对应的进程

```mysql
SELECT concat('KILL ', id, ';') FROM information_schema.processlist WHERE State = 'Locked';
```

然而杀掉进程后锁表情况依然存在，看来问题并不在这里，而且令我疑惑的是，除了考勤系统的库被锁，其他系统的为啥也存在比较多的锁表情况呢.....难道是sql的服务器出异常了，连接ssh看看。
先看df -f看下磁盘，哦豁，好家伙，空间满了！原来一上午在那里看的那些锁的语句啥的都做了无用功。
![1687159005252_9E856C09-6EAC-4036-A83C-CE6113650DB1.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/1687159005252_9E856C09-6EAC-4036-A83C-CE6113650DB1.png)

### 四、解决方案
通过清理根目录一些无用日志和文件等，清出来几十g的存储空间，还能抗一些日志，没办法，给的测试机资源有限。然后reboot重启服务器,oracle和mysql都是开机自启的，然后重启工程，数据插入更新的都没问题。


### 五、总结：
有时候遇到问题，我们往往会只关注问题报错的表面，然后尝试了多种方案都没能解决的时候，不妨静下心来思考下除了这个地方，其他层面有没有可能拖后腿，如磁盘，内存，网络等情况。要从多方面多维度思考问题，不要钻在牛角尖了拔不出来。
