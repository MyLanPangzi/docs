



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

1. 是什么
   1. 模拟优化器执行查询语句，从而分析语句或表结构性能瓶颈
2. 能干嘛
   1. 表的读取顺序
   2. 数据读取操作类型
   3. 哪些索引可以使用
   4. 哪些索引实际使用
   5. 表之间的引用
   6. 每张表有多少行被优化器查询
3. 用法
   1. Explain + SQL
4. 字段
   1. id
      1. id相同，从上至下顺序执行
      2. id不同，数字越大越先执行
   2. select_type
   3. table
   4. type
   5. possible_keys
   6. key
   7. key_len
   8. ref
   9. rows
   10. Extra

