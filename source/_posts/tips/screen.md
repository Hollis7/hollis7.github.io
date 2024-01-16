---
title: screen小技巧
categories:
  - linux
tags:
  - screen
  - linux
mathjax: true
---
<meta name="referrer" content="no-referrer"/>
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