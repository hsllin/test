文章转自 [面渣逆袭：二十二图、八千字、二十问，彻底搞定MyBatis！](https://mp.weixin.qq.com/s/O_5Id2o9IP4loPazJuiHng)
### mybatis介绍
Mybatis 是一个半 ORM（对象关系映射）框架，它内部封装了 JDBC，开发时只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建statement 等繁杂的过程。程序员直接编写原生态 sql，可以严格控制 sql 执行性能，灵活度高
**缺点**

-   SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求
    
-   SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库

### MyBatis使用过程？生命周期？

MyBatis基本使用的过程大概可以分为这么几步：

![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWcS8g3Glx7ibcJiaV20mkibH38zYoAYice4x5qubaPYibN1LQmR2J8iaU9eB8c4cQytPoBdvPhjd8WP104w/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Mybatis基本使用步骤

   1、 创建SqlSessionFactory
    
    可以从配置或者直接编码来创建SqlSessionFactory
    

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

```
2、 通过SqlSessionFactory创建SqlSession
SqlSession（会话）可以理解为程序和数据库之间的桥梁

```java
SqlSession session = sqlSessionFactory.openSession();
```

  3、 通过sqlsession执行数据库操作
  可以通过 SqlSession 实例来直接执行已映射的 SQL 语句：

```java
Blog blog = (Blog)session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);

```

更常用的方式是先获取Mapper(映射)，然后再执行SQL语句：
```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);

```

   4、 调用session.commit()提交事务
    
    如果是更新、删除语句，我们还需要提交一下事务。
    
  5、 调用session.close()关闭会话
    
    最后一定要记得关闭会话。
    

> **MyBatis生命周期？**

上面提到了几个MyBatis的组件，一般说的MyBatis生命周期就是这些组件的生命周期。

-   SqlSessionFactoryBuilder
    
    一旦创建了 SqlSessionFactory，就不再需要它了。因此 SqlSessionFactoryBuilder 实例的生命周期只存在于方法的内部。
    
-   SqlSessionFactory
    
    SqlSessionFactory 是用来创建SqlSession的，相当于一个数据库连接池，每次创建SqlSessionFactory都会使用数据库资源，多次创建和销毁是对资源的浪费。所以SqlSessionFactory是应用级的生命周期，而且应该是单例的。
    
-   SqlSession
    
    SqlSession相当于JDBC中的Connection，SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的生命周期是一次请求或一个方法。
    
-   Mapper
    
    映射器是一些绑定映射语句的接口。映射器接口的实例是从 SqlSession 中获得的，它的生命周期在sqlsession事务方法之内，一般会控制在方法级。
    

![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWcS8g3Glx7ibcJiaV20mkibH386NdiblLYzgRdZwMcWCEUrQAfdVowE30IIphZOQiabr8INIALh1T3eyyQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

MyBatis主要组件生命周期

当然，万物皆可集成Spring，MyBatis通常也是和Spring集成使用，Spring可以帮助我们创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到我们的 bean 中.

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/PMZOEonJxWcS8g3Glx7ibcJiaV20mkibH38DA0ib9oP02SwnnZTxuKfGsVeAtpIGr2p6ibTlqk52vhiandj7Lkkw7JuA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)


### 4.在mapper中如何传递多个参数？

![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWcS8g3Glx7ibcJiaV20mkibH38z1Z08E1G65e6PEu6yBUAaSZudMxS0N85wjDuwbjZ1kaUoX63JUnSoA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

mapper传递多个参数方法

**方法1：顺序传参法**

```java
public User selectUser(String name, int deptId);<select id="selectUser" resultMap="UserResultMap">   
select * from user    where user_name = #{0}
and dept_id = #{1}</select>
```

-   #{}里面的数字代表传入参数的顺序。
    
-   这种方法不建议使用，sql层表达不直观，且一旦顺序调整容易出错。
    

**方法2：@Param注解传参法**

```java
public User selectUser(@Param("userName") String name, int @Param("deptId") deptId);<select id="selectUser" resultMap="UserResultMap">    
select * from user    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

-   #{}里面的名称对应的是注解@Param括号里面修饰的名称。
    
-   这种方法在参数不多的情况还是比较直观的，（推荐使用）。
    

**方法3：Map传参法**

```java
public User selectUser(Map<String, Object> params);<select id="selectUser" parameterType="java.util.Map" resultMap="UserResultMap">    
select * from user    where user_name = #{userName} and dept_id = #{deptId}
</select>
```

-   #{}里面的名称对应的是Map里面的key名称。
    
-   这种方法适合传递多个参数，且参数易变能灵活传递的情况。
    

**方法4：Java Bean传参法**

```java
public User selectUser(User user);<select id="selectUser" parameterType="com.jourwon.pojo.User" resultMap="UserResultMap">    
select * from user    where user_name = #{userName} and dept_id = #{deptId
</select>
```

-   #{}里面的名称对应的是User类里面的成员属性。
    
-   这种方法直观，需要建一个实体类，扩展不容易，需要加属性，但代码可读性强，业务逻辑处理方便，推荐使用。（推荐使用）。
    

### 5.实体类属性名和表中字段名不一样 ，怎么办?

   第1种：通过在查询的SQL语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。
```java 
<select id="getOrder" parameterType="int" resultType="com.jourwon.pojo.Order">       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};</select> 
```
    
  第2种：通过resultMap  中的<result>来映射字段名和实体类属性名的一一对应的关系。
    
    ```java
    <select id="getOrder" parameterType="int" resultMap="orderResultMap"> select * from orders where order_id=#{id}</select>    <resultMap type="com.jourwon.pojo.Order" id="orderResultMap">    <!–用id属性来映射主键字段–>    <id property="id" column="order_id">    <!–用result属性来映射非主键字段，property为实体类属性名，column为数据库表中的属性–> <result property ="orderno" column ="order_no"/> <result property="price" column="order_price" />
    </reslutMap>
    ```
    

### 6.Mybatis是否可以映射Enum枚举类？

  Mybatis当然可以映射枚举类，不单可以映射枚举类，Mybatis可以映射任何对象到表的一列上。映射方式为自定义一个TypeHandler，实现TypeHandler的setParameter()和getResult()接口方法。
    
   TypeHandler有两个作用，一是完成从javaType至jdbcType的转换，二是完成jdbcType至javaType的转换，体现为setParameter()和getResult()两个方法，分别代表设置sql问号占位符参数和获取列查询结果。
    

### 7.#{}和${}的区别?

#{}和${}比较

-   #{}是占位符，预编译处理；${}是拼接符，字符串替换，没有预编译处理。
    
-   Mybatis在处理#{}时，#{}传入参数是以字符串传入，会将SQL中的#{}替换为?号，调用PreparedStatement的set方法来赋值。
    
-   #{} 可以有效的防止SQL注入，提高系统安全性；${} 不能防止SQL 注入
    
-   #{} 的变量替换是在DBMS 中；${} 的变量替换是在 DBMS 外
    

### 8.模糊查询like语句该怎么写?

![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWcS8g3Glx7ibcJiaV20mkibH38uZj72qg4ku30lcwCf3VyJ9p8ynyacaf6icWVjbBnabb42tv3Nyhs4vg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

concat拼接like

-   1 ’%${question}%’ 可能引起SQL注入，不推荐
    
-   2 "%"#{question}"%" 注意：因为#{…}解析成sql语句时候，会在变量外侧自动加单引号’ '，所以这里 % 需要使用双引号" "，不能使用单引号 ’ '，不然会查不到任何结果。
    
-   3 CONCAT(’%’,#{question},’%’) 使用CONCAT()函数，（推荐✨）
    
-   4 使用bind标签（不推荐）
    

```java
<select id="listUserLikeUsername" resultType="com.jourwon.pojo.User">&emsp;&emsp;<bind name="pattern" value="'%' + username + '%'" />&emsp;&emsp;select id,sex,age,username,password from person where username LIKE #{pattern}</select>
```

### 9.Mybatis能执行一对一、一对多的关联查询吗？

当然可以，不止支持一对一、一对多的关联查询，还支持多对多、多对一的关联查询。

![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWcS8g3Glx7ibcJiaV20mkibH38CcGxh0Pdq3Ymia7eicNJbjQfSXgIqGPgEY2ribibnES1wf11wwyfJJMPTA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

MyBatis级联

-   **一对一<association>**
    
    比如订单和支付是一对一的关系，这种关联的实现：
    

  实体类:
    
    ```java
    public class Order {   
     private Integer orderId;   
      private String orderDesc;   
      /**     * 支付对象     */    
      private Pay pay;    //……
      }
    ```
    
-   结果映射
    
    ```java
    <!-- 订单resultMap --><resultMap id="peopleResultMap" type="cn.fighter3.entity.Order">    <id property="orderId" column="order_id" />    <result property="orderDesc" column="order_desc"/>    <!--一对一结果映射-->    <association property="pay" javaType="cn.fighter3.entity.Pay">        <id column="payId" property="pay_id"/>        <result column="account" property="account"/>    </association></resultMap>
    ```
    
-   查询就是普通的关联查
    
    ```
        <select id="getTeacher" resultMap="getTeacherMap" parameterType="int">        select * from order o          left join pay p on o.order_id=p.order_id        where  o.order_id=#{orderId}    </select>
    ```
    

-   **一对多<collection>**
    
    比如商品分类和商品，是一对多的关系。
    

-   查询
    
    查询就是一个普通的关联查询
    
    ```
            <!-- 关联查询分类和产品表 -->        <select id="listCategory" resultMap="categoryBean">            select c.*, p.* from category_ c left join product_ p on c.id = p.cid        </select>  
    ```
    
-   实体类
    
    ```
    public class Category {    private int categoryId;    private String categoryName;      /**    * 商品列表    **/    List<Product> products;    //……}
    ```
    
-   结果映射
    
    ```
            <resultMap type="Category" id="categoryBean">            <id column="categoryId" property="category_id" />            <result column="categoryName" property="category_name" />                 <!-- 一对多的关系 -->            <!-- property: 指的是集合属性的值, ofType：指的是集合中元素的类型 -->            <collection property="products" ofType="Product">                <id column="product_id" property="productId" />                <result column="productName" property="productName" />                <result column="price" property="price" />            </collection>        </resultMap>
    ```
    

那么多对一、多对多怎么实现呢？还是利用<association>和<collection>，篇幅所限，这里就不展开了。