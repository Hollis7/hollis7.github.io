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

