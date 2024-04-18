---
title: Ubuntu小技巧
categories:
  - tips
tags:
  - ubuntu
mathjax: true
---
<meta name="referrer" content="no-referrer"/>

## 内容

- screen
- conda
- 常用命令

<!--more-->

## Screen

**新建一个screen会话**

screen -S <名字>

**退出当前screen会话**

键盘点击ctrl+a , 然后按d

**查看所有screen会话**

screen -ls

**进入（恢复）某一screen会话**

screen -r <会话序列号>

**关闭screen会话**

screen -X -S <序列号> quit

## conda

查看虚拟环境列表

~~~bash
conda env list
conda info --envs
~~~

删除环境

~~~bash
conda remove -n env_name --all
~~~

创建虚拟环境

~~~bash
conda create -n name python=3.8
~~~

激活虚拟环境

~~~bash
conda activate name
~~~

退出虚拟环境

~~~bash
conda deactivate
~~~

## 查看进程和杀死

~~~bash
top u name
pkill name
~~~

## 统计文件数

~~~bash
ls -1A | wc -l
~~~

## 查找文件

正则匹配

~~~bash
ls | grep '\.py$'
~~~

模糊匹配

~~~bash
ls | grep requi
~~~

~~~bash
(pytorch_yolo) ➜  yolov5-pytorch git:(main) ✗ ls | grep requi
requirements.txt
~~~

## Mysql使用

### 登录

~~~bash
# 使用root用户登录
mysql -u root -p
~~~

###  显示使用数据库列表

~~~bash
show databases;
use database_name;
~~~

### 建库与删库

~~~bash
create database seal_log character set utf8
drop database seal_log;
~~~

### 建表与删表

~~~bash
use seal_log;
create table 表名(字段列表);
drop table 表名;
~~~

~~~python
import pymysql
try:
    #创建与数据库的连接
    #参数分别是 指定本机 数据库用户名 数据库密码 数据库名 端口号 autocommit是是否自动提交（非常不建议，万一出问题不好回滚）
    db=pymysql.connect(host='localhost',user='xxx',password='xxx',database='seal_log',port=3309,autocommit=False)
    #创建游标对象cursor
    cursor=db.cursor()
    #使用execute()方法执行sql，如果表存在则删除
    cursor.execute('drop table if EXISTS logs')
    #创建表的sql
    sql='''CREATE TABLE IF NOT EXISTS logs
             (user_id INTEGER PRIMARY KEY,
              disk_speed_per REAL,
              cpu INTEGER,
              gpu INTEGER,
              pass_failed INTEGER,
              authorization INTEGER,
              transgression_number INTEGER,
              up_traffic INTEGER,
              down_traffic INTEGER,
              svm_cipher BLOB)'''
    cursor.execute(sql)
except Exception as e:
    print('创建表失败:', e)

finally:
    #关闭数据库连接
    db.close()
~~~

### 显示数据表的结构

~~~bash
describe logs;
~~~

## 端口打开与关闭

打开40000

~~~bash
sudo iptables -A INPUT -p tcp --dport 40000 -j ACCEPT
~~~

列出所有端口

~~~bash
sudo iptables -L INPUT --line-numbers
~~~

**删除特定编号的规则**： 根据上一步显示的编号，如果开放端口 40000 的规则是第 N 号，使用以下命令删除它：

~~~bash
sudo iptables -D INPUT N
~~~

