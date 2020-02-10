[TOC]

# 索引

## 简介

### 是什么

​	B树，排好序加快查找的数据结构

### 优势

1. 加快数据检索，减少磁盘IO
2. 提高排序效率，减少CPU计算成本 

### 劣势

1. 占用磁盘空间
2. 降低了表更新速度
3. 需时常维护，保持优化

### 类型

1. 单值索引

2. 唯一索引

3. 复合索引

4. 主键索引

5. 语法

   1. ```mysql
      CREATE [ONLINE | OFFLINE] [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name    [index_type]   
      ON tbl_name (key_part,...)    [index_option] ...
      key_part:    col_name [(length)] [ASC | DESC]
      index_option:    KEY_BLOCK_SIZE [=] value  
      				| index_type  
      				| WITH PARSER parser_name  
      				| COMMENT 'string'
      index_type:    USING {BTREE | HASH}
      ```

### 索引结构

1. BTREE
2.  HASH
3. RTREE
4. FULL-TEXT

### 适合建立索引的情况

1. 主键
2. Where条件
3. 外键
4. OrderBy字段
5. GroupBy字段
6. 优先考虑复合索引

### 不适合建立索引的情况

1. 经常增删改的表
2. 表记录少
3. 数据重复，分布平均

## 性能分析

### 优化器

### 性能瓶颈

1. CPU
2. IO
3. 服务器硬件

### Explain

1. **是什么**

   1. 模拟优化器执行查询语句，从而分析语句或表结构性能瓶颈

2. **能干嘛**

   1. 表的读取顺序
   2. 数据读取操作类型
   3. 哪些索引可以使用
   4. 哪些索引实际使用
   5. 表之间的引用
   6. 每张表有多少行被优化器查询

3. **用法**

   1. Explain + SQL

4. **字段**

   1. id

      1. id相同，从上至下顺序执行

      2. id不同，子查询ID会递增，数字越大越先执行

      3. 

         ```mysql
         explain select * from t1,t2,t3 where t1.id = t2.id and t3.id=t2.id;
         explain select id from t1 where id in (select id from t2 where id in (select id from t3 where t3.content = ''));
         explain select * from t2,(select * from t3 where t3.content = '') t3;
         
         ```

         

   2. select_type

      1. SIMPLE
      2. PRIMARY
      3. UNION

   3. table

   4. type

      1. system

         1. 表中只有一条记录

      2. const

         1. 单表查询，只有一条记录匹配

      3. **eq_ref**

         1. 多表查询中，用到了唯一性索引（主键/唯一索引），只能匹配一行

         2.  One row is read from this table for each combination of rows from the previous tables. Other than the [`system`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_system) and [`const`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_const) types, this is the best possible join type. It is used when all parts of an index are used by the join and the index is a `PRIMARY KEY` or `UNIQUE NOT NULL` index. 

         3.  [`eq_ref`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_eq_ref) can be used for indexed columns that are compared using the `=` operator. The comparison value can be a constant or an expression that uses columns from tables that are read before this table. In the following examples, MySQL can use an [`eq_ref`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_eq_ref) join to process *`ref_table`*: 

         4. ```mysql
            SELECT * FROM ref_table,other_table
              WHERE ref_table.key_column=other_table.column;
            
            SELECT * FROM ref_table,other_table
              WHERE ref_table.key_column_part1=other_table.column
              AND ref_table.key_column_part2=1;
            ```

            

      4. **ref**

         1. 查询条件用到了索引匹配，会匹配多行

         2.  All rows with matching index values are read from this table for each combination of rows from the previous tables. [`ref`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_ref) is used if the join uses only a leftmost prefix of the key or if the key is not a `PRIMARY KEY` or `UNIQUE` index (in other words, if the join cannot select a single row based on the key value). If the key that is used matches only a few rows, this is a good join type. 

         3.  [`ref`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_ref) can be used for indexed columns that are compared using the `=` or `<=>` operator. In the following examples, MySQL can use a [`ref`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_ref) join to process *`ref_table`*: 

         4. ```mysql
            SELECT * FROM ref_table WHERE key_column=expr;
            
            SELECT * FROM ref_table,other_table
              WHERE ref_table.key_column=other_table.column;
            
            SELECT * FROM ref_table,other_table
              WHERE ref_table.key_column_part1=other_table.column
              AND ref_table.key_column_part2=1;
            ```

            

      5. **range**

         1. 查询条件是范围匹配，in

         2.  Only rows that are in a given range are retrieved, using an index to select the rows. The `key` column in the output row indicates which index is used. The `key_len`contains the longest key part that was used. The `ref` column is `NULL` for this typ 

         3.  [`range`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_range) can be used when a key column is compared to a constant using any of the [`=`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_equal), [`<>`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_not-equal), [`>`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_greater-than), [`>=`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_greater-than-or-equal), [`<`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_less-than), [`<=`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_less-than-or-equal), [`IS NULL`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_is-null), [`<=>`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_equal-to), [`BETWEEN`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_between), [`LIKE`](https://dev.mysql.com/doc/refman/5.5/en/string-comparison-functions.html#operator_like), or [`IN()`](https://dev.mysql.com/doc/refman/5.5/en/comparison-operators.html#operator_in) operators: 

         4. ```mysql
            SELECT * FROM tbl_name
              WHERE key_column = 10;
            
            SELECT * FROM tbl_name
              WHERE key_column BETWEEN 10 and 20;
            
            SELECT * FROM tbl_name
              WHERE key_column IN (10,20,30);
            
            SELECT * FROM tbl_name
              WHERE key_part1 = 10 AND key_part2 IN (10,20,30);	
            ```

            

      6. **index**

         1. 从索引上检索数据，并未用于查找

         2. The `index` join type is the same as [`ALL`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_all), except that the index tree is scanned. This occurs two ways:

            - If the index is a covering index for the queries and can be used to satisfy all data required from the table, only the index tree is scanned. In this case, the `Extra`column says `Using index`. An index-only scan usually is faster than [`ALL`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_all) because the size of the index usually is smaller than the table data.
            - A full table scan is performed using reads from the index to look up data rows in index order. `Uses index` does not appear in the `Extra` column.

            MySQL can use this join type when the query uses only columns that are part of a single index.

      7. ALL

         1.  A full table scan is done for each combination of rows from the previous tables. This is normally not good if the table is the first table not marked [`const`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_const), and usually*very* bad in all other cases. Normally, you can avoid [`ALL`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_all) by adding indexes that enable row retrieval from the table based on constant values or column values from earlier tables. 

      8. ```mysql
   explain select * from (select * from t1  where id = 1) t;#system const
         explain select * from t1 inner join t2 on t1.id = t2.id;##eq_ref
         alter table t1 add index idx_t1_content(content); #ref
         explain select * from t1 where t1.content = '';#ref
         explain select * from t1 where id in (1,2,3);#range
         explain select id from t1;#index
         explain select * from t1;#all
         
         ```
      
         

   5. possible_keys

      1. 查询可能使用到的索引，一个或多个，但不一定被查询实际使用

      2. 理论用到，实际也用到了

      3. 理论用到，实际没用到

      4. 理论没用到，实际用到了

      5. 理论没用到，实际没用到

      6. ```mysql 
      alter table t1 add index idx_t1_content(content); #
         explain select * from t1 where t1.content = '';#possible_keys = key
         explain select * from t1 where id in (1,2,3);##possible_keys != key
         explain select id from t2;##possible_keys = null, key = primary
         explain select * from t2;#all is null
         ```
   
         

   6. key

      1. 实际使用到的索引，如果为NULL则未使用
   2.  It is possible that `key` will name an index that is not present in the `possible_keys` value. This can happen if none of the `possible_keys` indexes are suitable for looking up rows, but all the columns selected by the query are columns of some other index. That is, the named index covers the selected columns, so although it is not used to determine which rows to retrieve, an index scan is more efficient than a data row scan. 
      3.  For `InnoDB`, a secondary index might cover the selected columns even if the query also selects the primary key because `InnoDB` stores the primary key value with each secondary index. If `key` is `NULL`, MySQL found no index to use for executing the query more efficiently. 
      4. **InnoDB中，每一个辅助索引都保存了主键。如果主键与辅助索引一起出现，并不影响索引的覆盖。**
   
   7. key_len

      1. 索引可能使用最大字节数，同结果下，越短越好
   2. 计算规则
         1. int 4
         2. null 1
         3. 可变类型 2 varchar
         4. idx_age_name 5 + 13 = 18
   
   8. ref

      1. 哪些列或常量与所以中的列进行比较

      2. The `ref` column shows which columns or constants are compared to the index named in the `key` column to select rows from the table.

      3. ```mysql
   explain select * from t1,t2,t3 where t1.id = t2.id and t3.id=t2.id;
         
         1	SIMPLE	t1	index	PRIMARY	idx_t1_content	403		1	Using index
         1	SIMPLE	t2	eq_ref	PRIMARY	PRIMARY	4	hello.t1.id	1	""
         1	SIMPLE	t3	eq_ref	PRIMARY	PRIMARY	4	hello.t1.id	1	""
         ```
   
   9. rows
   
      1. 可能查询到的行数
   
   10. Extra
   
       1.  If you want to make your queries as fast as possible, look out for `Extra` values of `Using filesort` and `Using temporary`. 
   
       2. <span style="color:red;">Using filesort</span>
   
          1. **无法使用索引排序，必须使用额外的遍历**
   
          2.  MySQL must do an extra pass to find out how to retrieve the rows in sorted order. The sort is done by going through all rows according to the join type and storing the sort key and pointer to the row for all rows that match the `WHERE` clause. The keys then are sorted and the rows are retrieved in sorted order.  
   
          3. ```mysql
             explain select * from t2 order by content;# using filesort
             ```
   
       3. <span style="color:red;">Using temporary</span>
   
          1. **使用了临时表来保存结果，通常出现在分组以及排序条件与索引列顺序不符的情况下。**
   
          2.  To resolve the query, MySQL needs to create a temporary table to hold the result. This typically happens if the query contains `GROUP BY` and `ORDER BY` clauses that list columns differently. 
   
          3. ```mysql
             explain select * from t2 inner join t3 on t2.id = t3.id group by t3.content;#using temporary using filesort
             ```
   
       4. Using index
   
          1. **查询的数据从索引树中检索，不扫描真实的表**
   
          2. The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row. This strategy can be used when the query uses only columns that are part of a single index.
   
             For `InnoDB` tables that have a user-defined clustered index, that index can be used even when `Using index` is absent from the `Extra` column. This is the case if `type` is[`index`](https://dev.mysql.com/doc/refman/5.5/en/explain-output.html#jointype_index) and `key` is `PRIMARY`.
   
          3. ```mysql
             explain select id from t2;
             ```

## 案例分析

### 单表

1. 按查询顺序建立索引，范围之后全失效

```mysql
create table if not exists article(
    id int primary key auto_increment,
    author_id int not null ,
    category_id int not null ,
    views int not null ,
    comments int not null ,
    title varbinary(255) not null ,
    content text null
);
insert into article (author_id, category_id, views, comments, title, content)
values 
       (1,1,1,1,'1','1'),
       (2,2,2,2,'2','2'),
       (3,3,3,3,'3','3');
explain select  id,article.author_id from article where category_id=1 and comments>1 order by views limit 1;#all using filesort
alter table article add index idx_article_cid_comments_views(category_id,comments,views);#range using filesort
alter table article drop index idx_article_cid_comments_views;
alter table article add index idx_article_cid_views(category_id,views);#ref no filesort
```



### 双表

1. 左右连接，相反建索引

```mysql
CREATE TABLE IF NOT EXISTS `class` (
   `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
   `card` INT(10) UNSIGNED NOT NULL,
   PRIMARY KEY (`id`)
);
CREATE TABLE IF NOT EXISTS `book` (
    `bookid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
    `card` INT(10) UNSIGNED NOT NULL,
    PRIMARY KEY (`bookid`)
);
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
(FLOOR(1 + (RAND() * 20)));
explain select * from class left join book b on class.card = b.card;
explain select * from class right join book b on class.card = b.card;
alter table class add index idx_class_card(card);#left join book index right join book ref
alter table class drop index idx_class_card;
explain select * from book left join class on class.card = book.card;
explain select * from book right join class on class.card = book.card;
alter table book add index idx_book_card(card);#left join class index right join class ref
alter table book drop index idx_book_card;
```

### 三表

1. 小表驱动大表

```mysql
create table if not exists phone(
    phoneid int primary key auto_increment,
    card int not null
);
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20))),
                              (FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
                              (FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
                              (FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),
                              (FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20))),(FLOOR(1 + (RAND() * 20)));
explain select * from class inner join book on book.card = class.card inner join phone on phone.card = book.card;#using join buffer
explain select * from class left join book on book.card = class.card left join phone on phone.card = book.card;#ALL
alter table book add index idx_book_card(card);
alter table phone add index idx_phone_card(card);
explain select * from class left join book on book.card = class.card left join phone on phone.card = book.card;#ref ref 
```



## 索引失效

1. 全值匹配我最爱

   1. ```mysql
      create table  staffs(
          id int primary key auto_increment,
          name varchar(24) not null ,
          age int not null ,
          pos varchar(20) not null ,
          add_time timestamp not null  default CURRENT_TIMESTAMP
      );
      insert into staffs (name,age, pos)
      values
             ('z3',22,'manager'),
             ('July',23,'dev'),
             ('2000',23,'dev');
      alter table staffs add index idx_staffs_name_age_pos(name,age,pos);
      explain select * from staffs where name = '';#ref
      explain select * from staffs where name = '' and age = 10;#ref
      explain select * from staffs where name = '' and age = 10 and pos = '';#ref
      ```

2. 最佳左前缀法则

   1. ```mysql
      explain select * from staffs where age = 10 and pos = '';#ALL
      explain select * from staffs where name = '' and pos = '';#ref name index
      ```

3. 索引列上无计算

   1. ```mysql
      explain select * from staffs where name = 'July';#ref
      explain select * from staffs where LEFT(name, 4) = 'July';#ALL
      ```

4. 范围之后全失效

   1. ```mysql
      explain select * from staffs where name = '' and age > 10 and pos = '';#range name age index
      explain select * from staffs where name = '' and age >= 11 and pos = '';#range name age pos index
      ```

5. 尽量使用覆盖索引

   1. ```mysql
      explain select * from staffs where name = '' and age = 10 and pos = '';#Using where
      explain select name,age,pos from staffs where name = '' and age = 10 and pos = '';#Using where Using index
      explain select name,age,pos from staffs where name = '' and age > 10 and pos = '';#Using where Using index only use name index
      explain select name,age,pos from staffs where name = '' and pos = '';#Using where Using index only use name
      explain select name,age,pos from staffs where name = '' and age = 10;#Using where Using index use name age 
      ```

6. 使用不等于的时候会无法使用索引导致全表扫描

   1. ```mysql
      explain select * from staffs where name = '';#ref
      explain select * from staffs where name != '';#ALL 8.0是range
      ```

7. is null，is not null也无法使用索引

   1. ```mysql
      alter table staffs add column address varchar(20);
      explain select * from staffs where staffs.address is null;#ALL
      explain select * from staffs where staffs.address is not null;#ALL
      ```

   2. 字段允许为NULL时，is null会用到索引（8.0）

      1. ```mysql
         alter table staffs add index idx_staffs_address(address);
         explain select * from staffs where staffs.address is null;#ref
         ```

8. like 通配符开头会导致索引失效，变成全表扫描，可以使用覆盖索引

   1. ```mysql
      create table t_user(    id    int primary key auto_increment,    name  varchar(20),    age   int,    email varchar(20));
      insert into t_user (name, age, email)values('a',10,'123123@asdas.com'),('b',11,'1223@asdas.com'),('c',12,'12@asdas.com');
      alter table t_user add index idx_t_user_name_age(name,age);
      explain select * from t_user where name like '%b%';#ALL
      explain select * from t_user where name like '%b';#ALL
      explain select * from t_user where name like 'b%';#range
      explain select * from t_user where name like 'b%b%';#range
      explain select id from t_user where name like '%b%';#index Using index
      explain select name from t_user where name like '%b%';#index Using index
      explain select age from t_user where name like '%b%';#index Using index
      explain select name,age from t_user where name like '%b%';#index Using index
      explain select id,name,age from t_user where name like '%b%';#index Using index
      explain select id,name,age,email from t_user where name like '%b%';#ALL
      explain select * from t_user where name like '%b%';#ALL
      ```

9. 字符串不加单引号索引失效

10. ```mysql
    explain select * from t_user where name = '1';#ref
    explain select * from t_user where name = 1;#ALL
    ```

11. 少用or，用它连接时索引会失效

    1. ```mysql
       explain select * from t_user where name = '' or name = '1';#ALL
       ```

12. ![image-20191130181454750](.\images\image-20191130181454750.png)

## 面试题

```mysql

create table test(
    c1 char(10) not null ,
    c2 char(10) not null ,
    c3 char(10) not null ,
    c4 char(10) not null ,
    c5 char(10) not null
);
alter table test add index idx_c1234(c1,c2,c3,c4);
alter table test drop index idx_c1234;
explain select * from test where c1 = '';#ref c1
explain select * from test where c1 = '' and c2 = '';#ref c1 c2
explain select * from test where c1 = '' and c2 = '' and c3 = '';#ref c1 c2 c3
explain select * from test where c1 = '' and c2 = '' and c3 = '' and c4 = '';#ref c1 c2 c3 c4
explain select * from test where c2 = '' and c3 = '' and c4 = '' and c1 = '';#ref c1 c2 c3 c4
explain select * from test where c2 = '' and c3 = '' and c1 = '';#ref c1 c2 c3
explain select * from test where c1 = '' and c2 = '' and c3 > '' and c4 = '';#range c1 c2 c3
explain select * from test where c1 = '' and c2 = '' and c3 = '' and c4 > '';#range c1 c2 c3 c4
explain select * from test where c1 = '' and c2 = '' and c4 = '' order by c3;#ref c1 c2  #c3 not count into possible_keys use sort
explain select * from test where c1 = '' and c2 = '' order by c3;#ref c1 c2  #c3 not count into possible_keys use sort
explain select * from test where c1 = '' and c2 = '' order by c4;#ref c1 c2 using filesort
explain select * from test where c1 = '' and c5 = '' order by c2,c3;#ref c1 //c2 c3 not count into possible_keys use sort
explain select * from test where c1 = '' and c5 = '' order by c3,c2;#ref c1 using filesort
explain select * from test where c1 = '' and c2 = '' and c5 = '' order by c2,c3;#ref c1 c2 //c3 not count into possible_keys but use sort
explain select * from test where c1 = '' and c2 = '' and c5 = '' order by c3,c2;#ref c1 c2 //c2 is const,don`t affect sort
explain select * from test where c1 = '' and c4 = '' group by c2,c3;#ref c1 //c2 c3 use sort,强调：查询条件用到索引，中间列不能断开，必须也是过滤条件，不能是排序列
explain select * from test where c1 = '' and c4 = '' group by c3,c2;#ref c1 using temporary using filesort
```

1. 定值、范围还是排序，order by 是给个范围
2. group by基本上都需要排序，先排序再分组，会有临时表产生
3. ![image-20191130202833545](.\images\index-recommend)

# 查询截取分析

## 查询优化

1. 小表驱动大表

2. in exists
   1. A表，B表，A表记录大于B表记录使用in
   2. 否则使用exists SELECT * FROM A WHERE EXISTS (SELECT 1 FROM B WHERE A.id = B.id)
   
3. order by
   1. 避免使用filesort
   2. 符合索引最左前缀
   3. Where 使用索引的最左前缀定义为常量，order by使用索引
   4. 尽量在索引上完成排序操作
   5. 如果用到了filesort，调优sort_buffer，max_length_for_sort_data
   6. ![image-20191130230255047](.\images\order by)
   
4. group by
   1. 先排序后分组，符合索引最佳左前缀
   
   2. 同order by用到filesort时调优sort_buffer，max_length_for_sort_data
   
   3. where高于having，尽量在where完成限定
   
## 慢查询日志

   1. ```mysql
      show variables like '%slow_query_log%';
      set global slow_query_log =1;
      show variables like 'long_query_time';
      set global long_query_time = 1;
      show global status like '%Slow%';
      select sleep(2);
      ```
   
   2. 如果需要永久生效修改,my.cnf文件[mysqld]
   
   3. mysqldumpslow
   
      1. ```shell
         root@51644daabc6e:/var/lib/mysql# mysqldumpslow --help
         Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]
         
         Parse and summarize the MySQL slow query log. Options are
         
           --verbose    verbose
           --debug      debug
           --help       write this text to standard output
         
           -v           verbose
           -d           debug
           -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default
                         al: average lock time 
                         ar: average rows sent
                         at: average query time
                          c: count
                          l: lock time
                          r: rows sent
                          t: query time  
           -r           reverse the sort order (largest last instead of first)
           -t NUM       just show the top n queries
           -a           don't abstract all numbers to N and strings to 'S'
           -n NUM       abstract numbers with at least n digits within names
           -g PATTERN   grep: only consider stmts that include this string
           -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
                        default is '*', i.e. match all
           -i NAME      name of server instance (if using mysql.server startup script)
           -l           don't subtract lock time from total time	
         ```
      
      2. mysqldumpslow -s al -t 10 -g "left join" //根据平均锁时间，查询前十条带有左连接的记录

   ## Profile

1.    

   ```mysql
   show variables like 'profiling';
   set global profiling = 1;
   help 'show profile';
   select * from emp;
   select * from t_emp;
   select * from t_dept;
   select * from t_dept t inner join t_emp e on t.id = e.deptId;
   select * from emp group by id % 10 order by name limit 500000;
   select * from emp,dept limit 2000000;
   show profile ALL for query 11;
   show profile CPU,BLOCK IO for query 11;
   ```

   ![image-20191201091832962](.\images\show-profile)

# 锁机制

## 分类

1. 表锁

   1. MyISAM只支持表锁，不支持事务。

2. 行锁

   1. Innodb支持行锁，支持事务。

3. 读锁

   1. **针对同一份数据，可多个读操作，互不影响，阻塞写操作。**

   2. **读锁/写锁只能读取当前会话锁定的表，不能读取其他表**

   3. 案例

      ```mysql
      help 'lock';
      show open tables where In_use = 1;
      lock tables class read, book write;
      select * from class;
      select * from book;
      insert into book (bookid, card) values (123123,1231231);
      # insert into class (id, card)values (1111,123123123);#[HY000][1099] Table 'class' was locked with a READ lock and can't be updated
      # select * from emp;#[HY000][1100] Table 'emp' was not locked with LOCK TABLES
      unlock tables ;
      #另开一个会话，读book表，会阻塞
      ```

   4. ![image-20191201095431135](.\images\myisam-lock)

4. 写锁

   1. **排它锁，只有锁的拥有者可读可写，阻塞其他会话的操作。**

   2. 案例

      ```mysql
      lock tables book write ;
      select * from book;
      insert into book (bookid, card)values (9999,123123123);
      unlock tables ;
      ##另开一个会话
      select * from book;
      insert into book (bookid, card)values (666,666);
      ```

   3. ![image-20191201095630470](.\images\innodb-lock)

## 事务

1. ACID，事务是一组逻辑单元，要么全部执行成功，要么全部失败。
2. Atomicity 原子性
3. Consistent一致性
4. Isolation隔离性
5. Durable 持久性



## 并发带来的问题

1. 丢失更新
   1. 不支持事务的数据库会发生这种情况，也就是不支持锁的情况
2. 脏读
   1. 隔离级别为READ UNCOMMITED读取未提交的数据会发生脏读
3. 不可重复读
   1. 隔离级别为READ COMMITED读取已提交的数据会发生不可重复读
4. 幻读
   1. 只有SERIALIZABLE串行化隔离级别才能阻止幻读

## 事务隔离级别

1. REPEATABLE READ 可重复读 **innodb默认级别**，会发生幻读
2. READ COMMITED 读取已提交，会发生不可重复读，幻读
3. READ UNCOMMITED 读取未提交，会发生脏读，不可重复读，幻读
4. SERALIZABLE 串行化 不会发生任何并发问题，但会降低效率。
5. 案例演示
   1. ![image-20191201105943925](.\images\isolcation-1)

![image-20191201110045142](.\images\isolcation-2.png)

![image-20191201110219289](.\images\isolcation-3.png)

![image-20191201110247257](.\images\isolcation-4.png)

## 行锁变表锁

**更新数据的时候不要使用隐式的转换条件，不然会导致行锁变表锁**。

## 间隙锁

使用范围条件更新时，会产生间隙锁。

## 锁定一行

```mysql
begin;
select * from mylock where id = 1 for udpate;
commit;
```

## 优化建议

![image-20191201114005030](.\images\lock-suggest.png)



# 主从复制

## Master配置

```shell
echo -e 'server-id=1    #server实例的id  \n
log-bin=/var/lib/mysql/mysql-bin   #log-bin文件存储位置 \n
binlog_format=ROW  # 设置log-bin格式 STATEMENT   ROW  MIXED   \n

#可选的配置 \n
binlog-ignore-db=mysql  # 设置不要复制的数据库 \n
#binlog-do-db=xxx  # 设置需要复制的主数据库名字\n'  >> /etc/my.cnf.d/mysql-server.cnf

systemctl restart mysqld && sleep 20
mysql -uroot -pXiebo0409
CREATE USER 'slave'@'192.168.2.137' IDENTIFIED BY 'Xiebo0409';
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'192.168.2.137';
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;#记录position
exit

mysqldump -uroot -pXiebo0409 --all-databases --master-data > dbdump.db
scp -P 2222 ./dbdump.db root@192.168.2.137:/root/

```



## Slave配置

```shell
echo -e '
server-id=2    #server实例的id \n
relay-log=mysql-relay   #中继日志 \n
'  >> /etc/my.cnf.d/mysql-server.cnf

systemctl restart mysqld && sleep 20
mysql -uroot -pXiebo0409 < fulldb.dump
mysql -uroot -pXiebo0409
CHANGE MASTER TO MASTER_HOST='192.168.2.137',MASTER_USER='slave',MASTER_PASSWORD='Xiebo0409',MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=155;#修改MASTER_LOG_POS为Master的Position

START SLAVE;
SHOW SLAVE STATUS\G
STOP SLAVE;
```

# HA

