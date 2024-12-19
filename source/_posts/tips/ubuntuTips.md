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

## conda环境打包

1、 安装conda-pack

~~~bash
conda install -c conda-forge conda-pack
~~~

2、 打包虚拟环境

~~~bash
conda pack -n sf -o sf_output.tar.gz
~~~

3、 新机器上安装**打包的conda环境**

1. **复制压缩文件sf_output.tar.gz 到新的电脑环境**
2. 进入到conda的安装目录下：**/anaconda(或者miniconda)/envs/**，在该名目录下创建文件夹，复制sf_output.tar.gz 到文件夹中。
3. 解压conda环境：**tar -xzvf sf_output.tar.gz**
4. 使用conda env list查看虚拟环境
5. conda activate激活环境

## clash

命令开启

~~~bash
sudo bash start.sh
source /etc/profile.d/clash.sh
proxy_on
~~~

关闭

~~~bash
proxy_off
~~~

## python函数说明文档

查询函数库中的函数：

```bash
python3 -m pydoc -p 1234
```

## ubuntu免密登录坑

根据服务器日志，出现了以下重要信息，可能导致无法使用公钥进行免密登录：

```
Authentication refused: bad ownership or modes for directory /home/user/hdb
```

> 这是导致 SSH 公钥认证失败的主要原因，意味着 SSH 服务器拒绝了对目录 `/home/user/hdb` 的访问，原因是该目录的权限或所有权设置不正确。

服务器系统更新后，重新挂在了以前的用户目录，导致文件夹所属的组不是本人，权限也发生了变化，修改意见：

重新生成密钥对，然后更改相关权限：

~~~bash
chmod 700 /home/user/hdb
chmod 700 /home/user/hdb/.ssh
chmod 600 /home/user/hdb/.ssh/authorized_keys
chown -R hdb:hdb /home/user/hdb/.ssh
~~~

