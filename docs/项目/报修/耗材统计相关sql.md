
耗材统计和前端计算sql

```sql
    SELECT sum(a.sum_money),
       sum(b.MATERIAL_TOTAL_COST)
FROM
  (SELECT sum(sum_money) AS sum_money,
          id
   FROM
     (SELECT *
      FROM
        (SELECT nvl(d.num, 0)*nvl(d.PRICE, 0) AS sum_money,
                o.id
         FROM QYH_BU_REPAIR_MATERIAL_DATA d
         LEFT JOIN QYH_BU_REPAIR_ORDER o ON o.id=d.REPAIR_ORDER_ID
         LEFT JOIN QYH_BU_REPAIR_MATERIAL_INFO i ON i.code=d.code
         WHERE 1=1
         and  TO_CHAR(o.CREATE_TIME, 'yyyy-MM-dd') BETWEEN '2023-06-01' AND '2023-06-30'
         GROUP BY o.id,d.num,
                  d.price,
                  
                  d.code))
   GROUP BY id) a
JOIN QYH_BU_REPAIR_ORDER b ON a.id=b.id

```

按耗材维度统计

```sql
select sum(total_price) from (
    SELECT a.*,
       b.*
FROM
  (SELECT code,
       name,
       UPPER_CODE,
       nvl(sum(num), 0) AS sum_num,
       price AS sum_price,
       nvl(sum(num)*(price), 0) AS total_price
FROM
  (SELECT i.code,
          i.name,
          i.UPPER_CODE,
          sum(d.num) as num,
          i.PRICE
   FROM QYH_BU_REPAIR_MATERIAL_INFO i
   LEFT JOIN QYH_BU_REPAIR_MATERIAL_DATA d ON i.code=d.code 
   LEFT JOIN QYH_BU_REPAIR_ORDER o ON o.id=d.REPAIR_ORDER_ID 
   where 1=1 
   AND TO_CHAR(o.CREATE_TIME, 'yyyy-MM-dd') BETWEEN '2023-06-01' AND '2023-06-30'
      GROUP BY i.code,
            i.name,
            i.price,
            d.num,
            i.UPPER_CODE START WITH (i.code='-1' 
                                     OR i.UPPER_CODE='-1' ) CONNECT BY
   PRIOR i.code = i.UPPER_CODE)
 GROUP BY code,
         name,
         price,
         UPPER_CODE) a, 
  (SELECT i.code,
          i.name,
          LEVEL,
 sys_connect_by_path(i.code, '=>') AS total_code,
          sys_connect_by_path(i.name, '=>') AS total_name 
   FROM QYH_BU_REPAIR_MATERIAL_INFO i START WITH i.upper_code='-1' CONNECT BY
   PRIOR i.code = i.UPPER_CODE)b
WHERE a.code=b.code )

```
