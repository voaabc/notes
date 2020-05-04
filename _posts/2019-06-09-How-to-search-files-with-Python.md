---
layout: post
title: 如何用 Python 搜索文件
categories: Python 脚本
tags: Python Script
---

* content
{:toc}


## 范例

```python
import os
path = '/home/ubuntu01/test/script_project1_files/images' 

files = os.listdir(path)

print(files)

for f in files:
    if f.endswith('.png') and 'fish' in f:
        print('Look! I found this' + f)
```


> os.listdir() 方法用于返回指定的文件夹包含的文件或文件夹的名字的列表。这个列表以字母顺序。 它不包括 '.' 和'..' 即使它在文件夹中。只支持在 Unix, Windows 下使用。


### endswith()函数

* 描述：判断字符串是否以指定字符或子字符串结尾。

* 语法：str.endswith("suffix", start, end) 或 str[start,end].endswith("suffix")    
用于判断字符串中某段字符串是否以指定字符或子字符串结尾。
返回值为布尔类型（True,False）。suffix — 后缀，可以是单个字符，也可以是字符串，还可以是元组（"suffix"中的引号要省略，常用于判断文件类型）。start —索引字符串的起始位置。end — 索引字符串的结束位置。str.endswith(suffix)  start默认为0，end默认为字符串的长度len(str)

* 注意：空字符的情况。返回值通常为True

#### 程序示例
```python
str = "i love python"
print("1:",str.endswith("n")) 
print("2:",str.endswith("python"))
print("3:",str.endswith("n",0,6))# 索引 i love 是否以“n”结尾。
print("4:",str.endswith("")) #空字符
print("5:",str[0:6].endswith("n")) # 只索引 i love
print("6:",str[0:6].endswith("e"))
print("7:",str[0:6].endswith(""))
print("8:",str.endswith(("n","z")))#遍历元组的元素，存在即返回True，否者返回False
print("9:",str.endswith(("k","m")))
 
 
#元组案例
file = "python.txt"
if file.endswith("txt"):
    print("该文件是文本文件")
elif file.endswith(("AVI","WMV","RM")):
    print("该文件为视频文件")
else:
    print("文件格式未知")
```

#### 运行结果
```python
1: True
2: True
3: False
4: True
5: False
6: True
7: True
8: True
9: False
该文件是文本文件
```
