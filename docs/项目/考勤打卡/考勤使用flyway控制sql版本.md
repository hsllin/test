
## 1. 添加依赖  
  
- Flyway 依赖  
  
```xml  
<!-- 添加 flyway 的依赖 --><dependency>  
<groupId>org.flywaydb</groupId>  
<artifactId>flyway-core</artifactId>  
</dependency>  
```  
  
- 初始化表结构，需要操作数据库，因此引入数据库驱动以及数据源依赖，由于考勤用了shadingjdbc分库分表插件，所以不用再另外配置数据库驱动  

  
## 2. Flyway 知识补充  
  
1. Flyway 默认会去读取 `classpath:db/migration`，可以通过 `spring.flyway.locations` 去指定自定义路径，多个路径使用半角英文逗号分隔，内部资源使用 `classpath:`，外部资源使用 `file:`  
  
2. 如果项目初期没有数据库文件，但是又引用了 Flyway，那么在项目启动的时候，Flyway 会去检查是否存在 SQL 文件，此时你需要将这个检查关闭，`spring.flyway.check-location = false`  
  
3. Flyway 会在项目初次启动的时候创建一张名为 `flyway_schema_history` 的表，在这张表里记录数据库脚本执行的历史记录，当然，你可以通过 `spring.flyway.table` 去修改这个值  
  
4. Flyway 执行的 SQL 脚本必须遵循一种命名规则，`V<VERSION>__<NAME>.sql` 首先是 `V` ，然后是版本号，如果版本号有多个数字，使用`_`分隔，比如`1_0`、`1_1`，版本号的后面是 2 个下划线，最后是 SQL 脚本的名称。  
  
**这里需要注意：V 开头的只会执行一次，下次项目启动不会执行，也不可以修改原始文件，否则项目启动会报错，如果需要对 V 开头的脚本做修改，需要清空`flyway_schema_history`表，如果有个 SQL 脚本需要在每次启动的时候都执行，那么将 V 改为 `R` 开头即可**  
  
5. Flyway 默认情况下会去清空原始库，再重新执行 SQL 脚本，这在生产环境下是不可取的，因此需要将这个配置关闭，`spring.flyway.clean-disabled = true`  
  
## 3. application.yml 配置  
  
> 贴出我的 application.yml 配置  
  
```yaml  
spring:
  authentication : false
  #  mysql8.0  192.168.36.100  3306  root/123456
  #①进行水平分表(在单库中进行分表),已经实现，注意缩进问题
  shardingsphere:
    datasource:
      names: db
      db:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        #        192.168.36.100:3306/mc
        url: jdbc:mysql://192.168.2.37:3306/mc?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&rewriteBatchedStatements=true&serverTimezone=Asia/Shanghai
        username: root
        password: 37621040
    sharding:
      defaultDataSourceName: db
    props:
      sql:
        show: false
  flyway:
    enabled: true
    # 迁移前校验 SQL 文件是否存在问题
    validate-on-migrate: true
    # 生产环境一定要关闭
    clean-disabled: true
    # 校验路径下是否存在 SQL 文件
    check-location: false
    # 最开始已经存在表结构，且不存在 flyway_schema_history 表时，需要设置为 true
    baseline-on-migrate: true
    # 基础版本 0
    baseline-version: 0
    user: root
    password: 37621040
    url: jdbc:mysql://192.168.2.37:3306/mc?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&rewriteBatchedStatements=true&serverTimezone=Asia/Shanghai
#  datasource:
#    url: jdbc:mysql://192.168.2.37:3306/mc?useUnicode=true&useJDBCCompliantTimezoneShift=true&useLegacyDatetimeCode=false&rewriteBatchedStatements=true&serverTimezone=Asia/Shanghai
#    username: root
#    password: 37621040
#    type: com.zaxxer.hikari.HikariDataSource
```  
  
## 4. 测试  
  
### 4.1. 测试 1.0 版本的 SQL 脚本  
  
创建 `V1_0__INIT.sql`  
  
```mysql  
create table ly_mc_test  
(  
id varchar(60) not null comment 'id',  
tp varchar(60) comment '图片',  
dkjlid varchar(60) comment '打卡记录id',  
cjsj datetime comment '创建时间',  
lx char(10) comment '类型，1表示是管理员修改的图片',  
primary key (id)  
); 
```  
  
启动项目，可以看到日志输出：  
  
```java  
2023-07-12 16:55:59.897  INFO 26724 --- [           main] o.f.c.internal.license.VersionPrinter    : Flyway Community Edition 5.2.4 by Boxfuse
2023-07-12 16:55:59.979  INFO 26724 --- [           main] o.f.c.internal.database.DatabaseFactory  : Database: jdbc:mysql://192.168.2.37:3306/mc (MySQL 5.6)
2023-07-12 16:56:00.229  INFO 26724 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 1 migration (execution time 00:00.079s)
2023-07-12 16:56:01.270  INFO 26724 --- [           main] o.f.c.i.s.JdbcTableSchemaHistory         : Creating Schema History table: `mc`.`flyway_schema_history`
2023-07-12 16:56:01.579  INFO 26724 --- [           main] o.f.core.internal.command.DbBaseline     : Successfully baselined schema with version: 0
2023-07-12 16:56:01.647  INFO 26724 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema `mc`: 0
2023-07-12 16:56:01.661  INFO 26724 --- [           main] o.f.core.internal.command.DbMigrate      : Migrating schema `mc` to version 20221104.10001 - add
2023-07-12 16:56:01.830  INFO 26724 --- [           main] o.f.core.internal.command.DbMigrate      : Successfully applied 1 migration to schema `mc` (execution time 00:00.245s)
```  
  
检查数据库，发现创建了 2 张表，一张是 Flyway 依赖的历史表，另一张就是我们的 `ly_mc_test` 表  
  
<img src="http://static.xkcoding.com/spring-boot-demo/flyway/062903.jpg" alt="image-20200305105632047" style="zoom:50%;" />  
  
查看下 `flyway-schema-history` 表  
  
<img src="http://static.xkcoding.com/spring-boot-demo/flyway/062901.jpg" alt="image-20200305110147176" style="zoom:50%;" />  
 
  
## 参考  
  
1. [Spring Boot 官方文档 - Migration 章节](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#howto-execute-flyway-database-migrations-on-startup)  
2. [Flyway 官方文档](https://flywaydb.org/documentation/)