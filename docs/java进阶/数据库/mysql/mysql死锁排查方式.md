

#### 查看表使用
```mysql
show open tables where in_use > 0 ;
```
如果查询结果为空，那么说明表没在使用，说明不是锁表的问题
#### 查看当前运行的所有事务
```mysql
SELECT * FROM information_schema.INNODB_TRX;
```
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619161026.png)

#### 查询锁等待的对应关系
```mysql
SELECT * FROM information_schema.INNODB_LOCK_waits;
```
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619161104.png)

#### 查看当前出现的锁
```mysql
SELECT * FROM information_schema.INNODB_LOCKs;
```
![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619161120.png)


```mysql
show full PROCESSLIST
```

![image.png](https://images-lin.oss-cn-guangzhou.aliyuncs.com/images/20230619160818.png)

#### kill掉事务
kill上面查出来的id

### 批量 kill processlist


```mysql
SELECT concat('KILL ', id, ';') FROM information_schema.processlist WHERE State = 'Locked';
```