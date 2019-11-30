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

12. ![image-20191130181454750](C:\Users\Administrator\Documents\Typro\images\image-20191130181454750.png)

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
3. ![image-20191130202833545](C:\Users\Administrator\Documents\Typro\images\index-recommend)

# 查询截取分析