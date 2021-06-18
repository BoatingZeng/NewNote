## 数据库范式

https://www.zhihu.com/question/24696366/answer/29189700
https://www.zhihu.com/question/24696366/answer/29049568

1. 第一范式(1NF)：一范式就是属性不可分割。
2. 第二范式(2NF)：二范式就是要有主键，要求其他字段都依赖于主键。
3. 第三范式(3NF)：三范式就是要消除传递依赖，方便理解，可以看做是“消除冗余”，就是各种信息只在一个地方存储，不出现在多张表中。

## 常用函数

1. substring_index(str,delim,count)

      str:要处理的字符串

      delim:分隔符

      count:计数

http://blog.csdn.net/wolinxuebin/article/details/7845917

2. concat('a','b','c') = 'abc';
3. `IFNULL(fieldName, value)`，如果这个`fieldName`字段的值是`null`，则用`value`代替
4. `DATE_FORMAT(date, format)`，格式化时间。
5. `COUNT()`，`COUNT(column_name)`会计数`column_name`不为`NULL`的行，`COUNT(*)`会计数所有行。
6. `REPEAT('a', 3) = 'aaa' `; 重复字符串
7. `LENGTH('aaa') = 3`; 计算字符串长度

## 导出导入

1. 导出整个数据库
```
    mysqldump -u 用户名 -p 数据库名 > 导出的文件名
    mysqldump -u root -p test > test.sql
```
2. 导出一个表
```
    mysqldump -u 用户名 -p 数据库名 表名 > 导出的文件名
    mysqldump -u root -p test users > test_users.sql
```
3. 导出一个数据库结构
`mysqldump -u root -p -d --add-drop-table smgp_apps_wcnc > d:\wcnc_db.sql`
* -d：没有数据
* --add-drop-table：在每个create语句之前增加一个drop table
4. 导入数据库(常用source命令)
  1. 进入mysql数据库控制台，如：mysql -u root -p
  2. mysql>use 数据库  **一定要记得先选择数据库**
  3. mysql>source d:\wcnc_db.sql
5. 只导出数据
```
    mysqldump -t job -u root -p > test.sql
```

远程： 加 --host
http://dev.mysql.com/doc/refman/5.7/en/mysqldump.html

## 基本

### 语句
**SQL语句的顺序问题。**
http://www.cnblogs.com/qanholas/archive/2010/10/24/1859924.html
1. `USE dbName`选择`dbName`数据库
2. `SELECT tb.fieldName FROM tableName tb`选择`tableName`表的`fieldName`字段，`tb`用于给`tableName`起别名。
3. `IN`，`WHERE column_name IN (value1,value2)`，`IN`表示可能的取值，即相当于`WHERE column_name=value1 OR column_name=value2`
4. `LIKE`，`WHERE column_name LIKE pattern`，`LIKE`用于指定模式，`pattern`就是模式
5. `AS`，`origin AS new`，用于起别名，用`new`表示`origin`，`AS`可以省略，中间用空格
6. `UNION`，用于合并两个或多个`SELECT`语句的结果，`UNION`选取不同的值，`UNION ALL`允许选择重复的值，结果集中的列名总等于第一个`SELECT`语句的列名
7. `JOIN`，跟平时用的两个表`WHERE`结合查询是一个意思，会先生成一个临时表。`JOIN`的时候，表应该从小到大排列，在`FROM`里也是，大表放在结合操作的右侧。`JOIN`操作应该在过滤操作前面。

### Stored Procedure (存储过程)

1. 执行`loopinsert`就可以循环插入

```sql
create procedure loopinsert() 
begin 
declare num int; 
set num=1; 
    while num < 10 do 
    insert into t_tag(tag_name, tag_desc) values(concat("tag", num), concat("tag", num)); set num=num+1;
    end while;
end
```

### 关于SELECT 0
语句`SELECT 0 AS x, field1 FROM tb`， tb表中是没有`x`这个字段的，但是希望查询结果中有这个字段，所以就通过`SELECT 0 AS x`来添加了，并且结果中的`x`字段会是`0`值。

具体例子：http://stackoverflow.com/questions/13219472/why-SELECT-0-instead-of-SELECT
。这个例子是为了确认`B`表中到底有没有`a=1`和`a=2`的行。

### ORDER BY
* 对于InnoDB：https://segmentfault.com/a/1190000015987895

1. where和order by使用的索引不同时，只能用其中一个
2. 如果不用where，查了排序字段和主键之外的字段，那还是可能发生filesort
3. 如果对排序字段用where，比如限制个范围，如果范围比较大，那还是可能发生filesort

实例：
```sql
-- 表结构
CREATE TABLE `t_test` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  `idx1` int(10) unsigned NOT NULL,
  `idx2` int(10) unsigned NOT NULL,
  PRIMARY KEY (`id`),
  KEY `i1` (`idx1`),
  KEY `i2` (`idx2`)
) ENGINE=InnoDB AUTO_INCREMENT=18500 DEFAULT CHARSET=utf8;

-- 存储过程，插10w条数据。测试时太慢，插了20620条就放弃了
CREATE DEFINER=`root`@`localhost` PROCEDURE `insert_test`()
BEGIN
	DECLARE v INT DEFAULT 1;
    WHILE v<100000
		DO
        INSERT INTO t_test VALUES(v,v,v,v);
        SET v=v+1;
	END WHILE;
END

-- 对应上面几种情况的查询

-- 情况1
select * from t_test where idx1 < 500 order by idx2 desc; -- where和order by所用索引不同，filesort

-- 情况2
select * from t_test order by idx1 desc; -- filesort
select idx1, id from t_test order by idx1 desc; -- 只查主键和排序索引，不用filesort
select idx1, id, idx2 from t_test order by idx1 desc; -- filesort，即使全是索引

-- 情况3
select idx1 from t_test where idx1 > 500 order by idx1 desc; -- 范围很大，但是因为只查了索引字段，不用filesort，这个其实跟情况2的没where只查索引差不多
select * from t_test where idx1 > 500 order by idx1 desc; -- 范围很大，用filesort
select * from t_test where idx1 < 500 order by idx1 desc; -- 范围很小，不用filesort
```

## 例子

### 例1
```sql
SELECT tcpr.rel_id, tcpr.seq, tp.code, tp.title, tp.creation_date, tp.lASt_update_date
FROM t_product tp, t_category_product_relation tcpr
WHERE tp.code=tcpr.product_code AND tcpr.category_code='cy_yinyue'
ORDER BY tcpr.seq ASC
```

这个语句查询了两个表，`t_product`和`t_category_product_relation`，选择了
`tcpr`表的`rel_id`、`seq`字段，`tp`表的`code`、`title`、`creation_date`、`lASt_update_date`字段。

关于`WHERE`的部分，是先从`tcpr`表获取`category_code='cy_yinyue'`的部分，然后从`tp`表获取`tp.code=tcpr.product_code`的部分。
关于`ORDER BY`部分，结果按照`tcpr.seq`升序排序。

(注:`tp.code`和`tcpr.product_code`记录的数据是一样的，就是`pcode`，下面的例子也是)

### 例2
```sql
SELECT tp.code, tp.title, tp.creation_date, tp.lASt_update_date
FROM t_product tp 
WHERE tp.code NOT IN (
    SELECT product_code FROM t_category_product_relation
    WHERE  category_code = 'cy_huihua'
)
AND tp.code LIKE '通配'
ORDER BY tp.code
```

先从`t_category_product_relation`表中选择符合`category_code = 'cy_huihua'`的记录的`product_code`字段，这些`product_code`用作`IN`所需的数组。这样就排除了`category_code = 'cy_huihua'`的`tp.code`。

### 例3
```sql
    SELECT resource_code, seq, 0 AS i 
    FROM t_product_resource_relation t
    WHERE t.product_code = 'pcode' 
    AND t.seq = (SELECT CASE seq WHEN 0 THEN - 1 ELSE seq - 1 END
        FROM t_product_resource_relation
        WHERE product_code = 'pcode'
        AND resource_code = 'rcode')
UNION
    SELECT resource_code, seq, 1 AS i 
    FROM t_product_resource_relation t
    WHERE t.product_code = 'pcode'
    AND t.seq = (SELECT seq + 1 
        FROM t_product_resource_relation
        WHERE product_code = 'pcode'
        AND resource_code = 'rcode')
```

查询上/下一条资源记录，查询结果的`0`和`1`应该就是分别表示上一集和下一集。先用`pcode`和`rcode`查到当前集，然后当前集的`seq`减一和加一分别就是上一集和下一集的`seq`，然后用用上一集和下一集的`seq`找到它们的`rcode`。

### 例4
```sql
    SELECT tpr.resource_code, t.product_code, t.seq, 0 AS i
    FROM t_category_product_relation t, t_product_resource_relation tpr
    WHERE t.product_code = tpr.product_code
    AND t.category_code = 'ccode'
    AND t.seq = (SELECT CASE seq WHEN 0 THEN - 1 ELSE seq - 1 END
        FROM t_category_product_relation
        WHERE category_code = 'ccode'
        AND product_code = 'pcode')
UNION
    SELECT tpr.resource_code, t.product_code, t.seq, 1 AS i
    FROM t_category_product_relation t, t_product_resource_relation tpr
    WHERE t.product_code = tpr.product_code
    AND t.category_code = 'ccode'
    AND t.seq = (SELECT seq + 1
        FROM t_category_product_relation
        WHERE category_code = 'ccode'
        AND product_code = 'pcode')
```

根据分类编码查询上/下一条资源记录，适用于查询每个作品仅包含1集资源的情况。为什么只有一集还要去查找上一集和下一集？？？

```sql
select * from post where content like '%$中文%' and date > '两个月内时间'
```
由于***id是自增***的，用下面的方式效率应该更高点
```sql
select p.* 
from post as p
     ,(select id from post where  date > '两个月内时间'  order by id limit 1) as t
where content like '%$中文%' and p.id>=t.id 
order by p.id desc limit 0,30;
```
先取ID，然后用ID做顺序扫描的写法是个好方法。
当然前提是 自增ID 和 date是单调递增的。

通过主键顺序搜索映射到磁盘上就是顺序读。10万行1k的数据相当于100M，基本是秒级完成。
再加上like的匹配，相对于作业类型的SQL属于可接受范围。

## 事务
```sql
start transaction;
INSERT INTO `test`.`repo_table` (`repo_id`, `repo_name`) VALUES ('17', 'kl');
INSERT INTO `test`.`repo_table` (`repo_id`, `repo_name`) VALUES ('18', 'mn');
commit; //或者rollback
```
相当于上保险，如果其中有语句ERRO，则执行`rollback`，则所有语句都不实际生效。但是一旦执行`commit`，则语句都会生效。

### mysql.js中的用法
```js
connection.beginTransaction(function(err) {
  if (err) { throw err; }
  connection.query('INSERT INTO posts SET title=?', title, function(err, result) {
    if (err) {
      return connection.rollback(function() {
        throw err;
      });
    }

    var log = 'Post ' + result.insertId + ' added';

    connection.query('INSERT INTO log SET data=?', log, function(err, result) {
      if (err) {
        return connection.rollback(function() {
          throw err;
        });
      }
      connection.commit(function(err) {
        if (err) {
          return connection.rollback(function() {
            throw err;
          });
        }
        console.log('success!');
      });
    });
  });
});
```
其实就是如果query抛出错误，就rollback，这样之前的query都不会执行。都没问题才commit。

## KEY
http://stackoverflow.com/questions/10908561/mysql-meaning-of-primary-key-unique-key-and-key-when-used-together

在MYSQL中Key就是Index。不完全相同，但是创建Key一定会创建Index。比如要创建PK就不能用create index。对于InnoDB，创建表时，如果不指定PK，就会创建隐藏的PK。
Unique Key 就是 Unique Index。Unique约束和Unique Key、Unique Index的结果是一样的。不过PK不能通过INDEX创建。

### PRIMARY （主键）

如果你用InnoDB，而且不需要特殊的聚簇索引，一个好的做法就是使用代理主键(surrogate key)——独立于你的应用中的数据。最简单的做法就是使用一个AUTO_INCREMENT的列，这会保证记录按照顺序插入，而且能提高使用primary key进行连接的查询的性能。应该尽量避免随机的聚簇主键，例如，字符串主键就是一个不好的选择，它使得插入操作变得随机。

### FOREIGN KEY （外键）
http://www.cnblogs.com/mydomain/archive/2011/11/10/2244233.html
建立外键的表叫子表，被reference的表叫父表。注意`ON UPDATE`和`ON DELETE`事件的设置，默认是`RESTRICT`。默认状态下，子表不可添加父表中没有关联的记录，父表不能删除与子表关联的记录。**也就是子表中有的，父表一定要有。**
