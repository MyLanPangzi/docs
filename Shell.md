# 文本处理工具

## Grep

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

1. -d delimiter 定界符，拆分符，默认是制表符
2. -f feilds 字段集合，常搭配d一起使用
3. -c chars 截取前几个字符
4. Range
   1. N 截取前几个
   2. N- 从几开始到结束
   3. N-M N到M 闭区间
   4. -M 从1开始到M，闭区间