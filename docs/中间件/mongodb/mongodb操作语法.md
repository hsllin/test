###创建索引
#### ensureIndex() 方法

MongoDB使用 `ensureIndex()` 方法来创建索引。

###### 语法

ensureIndex()方法基本语法格式如下所示：

```sql
db.COLLECTION_NAME.ensureIndex({KEY:1})
```

语法中 Key 值为你要创建的索引字段，1为指定按升序创建索引，如果你想按降序来创建索引指定为-1即可。

###### 实例

```
db.col.ensureIndex({"title":1})
```

ensureIndex() 方法中你也可以设置使用多个字段创建索引（关系型数据库中称作复合索引）。

```
db.col.ensureIndex({"title":1,"description":-1})
```

#### 对索引的操作

-   查看集合索引
    

```
db.col.getIndexes()
```

-   查看集合索引大小
    

```
db.col.totalIndexSize()
```

-   删除集合所有索引
    

```
db.col.dropIndexes()
```

-   删除集合指定索引
    

```
db.col.dropIndex("索引名称")
```

### 聚合 - Aggregation Pipline

类似于将SQL中的group by + order by + left join ... 等操作管道化。

#### 常规使用

-   图例理解![图片](https://mmbiz.qpic.cn/mmbiz_png/tuSaKc6SfPpboR03WSzoQnzssLX1C2PTTh01IPzdt8lgGoH0CNNZqc8QOcX8u04rZZF7gCibZ846aflAfgubzFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)
    
-