# 文本处理工具

## Grep

**多文件搜索**

1. --color=auto
   1. alias grep="grep --color=auto"
   2. /etc/bashrc    append previous line
2. -n 行号
3. -i 忽略大小写
4. ^开头 $结尾
5. -v 反选
6. -B before 前几行
7. -A after 后几行
8. -C context 上下文几行
9. -w word统计单词
10. -o 只打印关键字，可配合 wc 使用

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

## Uniq

**多文件去重，只去除连续重复的行**

1. -i 忽略大小写
2. -u unique 只打印唯一的行
3. -c 统计重复次数
4. -d 只显示重复的行
5. -D duplicate 显示所有重复的行
6. -f 避免比较前n个制表符
7. -s 跳过前n个字符比较

## tee

**多文件追加，从输入流打印到文件以及输出流**

1. -a 追加

