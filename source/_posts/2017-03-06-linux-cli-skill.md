---
title: linux命令行的一些技巧和脚本收集
date: 2017-03-06 23:54:52
tags:
  - linux
  - cli
  - 技巧
---
# 一些技巧

|技巧|说明|
|----|----|
| sudo !! | 以root权限执行上一条命令（注意上一条命令的内容，以免发生意外）|
| !+数字 | 执行history中相应编号的命令 |
| ^foo^bar | 将上一条命令中的第一个字符串"foo"替换成“bar"后执行 |
| !!:gs/foo/bar | 将上一条命令中的字符串“foo"全部替换成”bar"，然后执行 |
| C-x e | 打开编辑器编辑命令，这个编辑器是环境变量中EDITOR的值 |
| Alt+./ESC+. | 将最近一条命令的参数输出——不是作为结果打印出来，而是在终端提示符后面输出，还可以进行编辑 |

# 一些命令

## 查找src目录下有printf这个内容的cpp文件
```bash
grep src/*.cpp -e printf | cut -d ':' -f1 | uniq
```

## 按文件大小增序打印出当前目录下的文件名及其文件大小(单位字节）
```bash
ls -l | sed '1d' | sort -n -k5 | awk '{printf "%15s %10s\n", $9,$5}'
```

## 将当前目录下最大的5个文件移动到 ~/five-biggest/ 这个目录下
```bash
ls -l | sed '1d' | sort -n -r -k5 | head -n 5 | xargs -I {} mv {} ~/five-biggest/
```

## 删除文件中的空行
```bash
cat a.txt | sed -e '/^$/d'
```
## 列出用的最多的命令
``` bash
history -1 | awk '{a[$2]++}END{for(i in a){print a[i] " " i}}' | sort -rn | head
```
