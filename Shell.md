[TOC]

# 文本处理工具

## Grep

**多文件搜索**

1. ![image-20191207151427657](images\grep.png)
2. ![image-20191207151837327](images\grep-案例.png)

## Cut

**多文件剪切**

1. -d delimiter 定界符，拆分符，默认是制表符
2. -f feilds 字段集合，常搭配d一起使用
3. -c chars 截取前几个字符
4. Range
   1. N 截取前几个
   2. N- 从几开始到结束
   3. N-M N到M 闭区间
   4. -M 从1开始到M，闭区间
5. ![image-20191207151923797](images\cut.png)
6. ![image-20191207152138085](images\cut-案例.png)
7. 课堂练习![image-20191207152233458](images\课堂练习1.png)

## Sort

**多文件排序**

1. -u 去重
2. -r 降序
3. -R 随机排序
4. -n 以数字排序
5. -t 分隔符
6. -k第几列
7. -b 忽略前导空格
8. -d 只考虑空白与字母
9. 
10. ![image-20191207152332655](images\sort.png)
11. ![image-20191207152403207](images\sort-案例.png)

## Uniq

**多文件去重，只去除连续重复的行**

1. -i 忽略大小写
2. -u unique 只打印唯一的行
3. -c 统计重复次数
4. -d 只显示重复的行
5. -D duplicate 显示所有重复的行
6. -f 避免比较前n个制表符
7. -s 跳过前n个字符比较
8. ![image-20191207152453365](images\uniq.png)

## Tee

**多文件追加，从输入流打印到文件以及输出流**

1. -a 追加
2. ![image-20191207152521038](images\tee.png)

## Diff

1. ![image-20191207152554110](images\diff1.png)
2. ![image-20191207152643831](images\diff2.png)
3. ![image-20191207152722134](images\diff-normal.png)
4. ![image-20191207152828419](images\diff-context.png)
5. ![image-20191207152922352](images\diff-merge.png)
6. ![image-20191207153028361](images\diff-dir.png)
7. ![image-20191207153136443](images\diff-patch.png)

## Paste

1. ![image-20191207153212483](images\paste.png)

## Tr

1. 搜索替换
2. 删除
3. ![image-20191207153315132](images\tr-1.png)
4. ![image-20191207153349339](images\tr-2.png)
5. ![image-20191207153431547](images\tr-3.png)
6. ![image-20191207153754499](images\课堂练习2.png)
7. ![image-20191207153830961](images\课堂练习3.png)

# Bash

## 常见快捷键

![image-20191207143711329](images\shorthands.png)

## 通配符

![image-20191207143859088](images\通配符.png)

## 引号

![image-20191207144848958](images\引号.png)

![image-20191207145304472](images\引号案例.png)