---
title: mysql的一些函数
date: 2015-09-23 23:10:52
tags:
  - mysql
  - function
  - 笔记
category: mysql
---

# 字符串函数

concat 连接字符串
RTrim 删除右边的空格
upper 转为大写
left 返回字符串左边的字符
length 返回字符串的长度
locate 查找子字符串
lower 转为小写
LTrim 去掉左边的空格
Right 返回串右边的字符
soundex 返回串的soundex值 将任何文本转为拼音
substring 返回子串的字符

# 时间和日期处理函数

addDate()  增加一个日期（天，周等）
addTime()  增加一个时间（时，分等）
CurDate()  返回当前日期
CurTime()  返回当前时间
Date()  返回日期时间的日期部分
DateDiff() 计算两个日期之差
Date_Add() 高度灵活的日期运算函数
Date_Format() 返回一个格式格的日期或时间串
Day() 返回一个日期的天数部分
DayOfWeek() 返回星期几
Hour() 返回小时
Minute() 返回分钟
Mouth() 返回月
Now() 返回当前日期和时间
Second() 返回秒
Time() 返回时间部分
Year() 返回年

# 常用的数值处理函数

abs 绝对数
cos 余玄
exp 指数
mod 余数
pi 圆周率
rand 随机数
sin 正玄
sqrt 平方根
tan 角度的正切

# 聚集函数
avg 平均值
count 行数
max 最大值
min 最小值
sum 和
