---
layout: post
title: 用 Python 实现文件查找
categories: Python 脚本 图形界面
tags: Python Script GUI
---

* content
{:toc}


## 利用内置函数实现文件查找



### 功能
> 返回用户输入的文件的绝对路径


### 设计思路


1. 用户输入在哪个盘进行查找
2. 遍历此盘文件，若为目标文件则输出
3. 无此文件，则输出错误


### 实验代码
```python
#查找某个目录下的目标文件
import os       #引入操作系统模块
import sys      #用于标准输入输出

def search(path,name):

    for root, dirs, files in os.walk(path):  # path 为根目录
        if name in dirs or name in files:
            flag = 1      #判断是否找到文件
            root = str(root)
            dirs = str(dirs)
            return os.path.join(root, dirs)
    return -1


path = input('请输入您要查找哪个盘中的文件（如：D:\\\）')
print('请输入您要查找的文件名：')
name = sys.stdin.readline().rstrip()  #标准输入,其中rstrip()函数把字符串结尾的空白和回车删除
answer = search(path,name)
if answer == -1:
    print("查无此文件")
else:
    print(answer)

```

### 运行结果展示

无此文件
```sh
(test01) ubuntu01@ubuntu01:~/test9$ pwd
/home/ubuntu01/test9
(test01) ubuntu01@ubuntu01:~/test9$ ll
total 12
drwxrwxr-x  2 ubuntu01 ubuntu01 4096 Dec 24 09:41 ./
drwxr-xr-x 16 ubuntu01 ubuntu01 4096 Dec 24 09:40 ../
-rw-rw-r--  1 ubuntu01 ubuntu01  493 Dec 24 09:41 ex29.py
-rw-rw-r--  1 ubuntu01 ubuntu01    0 Dec 24 09:40 text1.txt
-rw-rw-r--  1 ubuntu01 ubuntu01    0 Dec 24 09:40 text3.txt
(test01) ubuntu01@ubuntu01:~/test9$ python ex29.py 
请输入您要查找哪个盘中的文件（如：D:\\）/home/ubuntu01/test9
请输入您要查找的文件名：
yeeeee.txt
查无此文件
```
有此文件
```
(test01) ubuntu01@ubuntu01:~/test9$ python ex29.py 
请输入您要查找哪个盘中的文件（如：D:\\）/home/ubuntu01/test9
请输入您要查找的文件名：
text1.txt
/home/ubuntu01/test9/[]
```


> os.walk() 方法用于通过在目录树中游走输出在目录中的文件名，向上或者向下。

> readline() 方法用于从文件读取整行，包括 "\n" 字符。如果指定了一个非负数的参数，则返回指定大小的字节数，包括 "\n" 字符。


## 队列实现文件查找

### 设计思路
```python
定义队列 ALLFiles 存储所有文件

while ALLFiles 不为空
    if pop 为目录
        then 将目录内所有文件入队
        elesif pop 为文件
            then if 为目标文件
                    then break
    end
输出路径
```

### 实验代码
```python
#查找某个目录下的目标文件
import os       #引入操作系统模块
import sys      #用于标准输入输出
import easygui as g     #引入图形用户界面

def search(path1,name):
    Allfiles = []           #创建队列
    Allfiles.append(path1)

    while len(Allfiles) != 0:    #当队列中为空的时候跳出循环
        path =Allfiles.pop(0)    #从队列中弹出首个路径
        if os.path.isdir(path):  #判断路径是否为目录
            ALLFilePath =os.listdir(path)    #若是目录，遍历将里面所有文件入队
            for line in ALLFilePath:
                newPath =path +"\\"+line   #形成绝对路径
                Allfiles.append(newPath)
        else:   #如果是一个文件，判断是否为目标文件
            target = os.path.basename(path)
            if target == name:
                return path
    return -1

path = g.enterbox(msg='请输入文件目录(如:D:DEV)')
name = g.enterbox(msg='请输入您要查找的文件名：')
answer = search(path,name)
if answer == -1:
    g.msgbox("查无此文件",'查找错误')
else:
    g.msgbox(answer,'返回路径')
```

> pop() 函数用于移除列表中的一个元素（默认最后一个元素），并且返回该元素的值。

> append() 方法用于在列表末尾添加新的对象。


