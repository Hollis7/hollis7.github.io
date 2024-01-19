---
title: git小技巧
categories:
  - tips
tags:
  - git
mathjax: true
---
<meta name="referrer" content="no-referrer"/>

## git设置代理

<!--more-->

### git中设置代理

~~~bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
~~~

### git取消代理

~~~bash
git config --global --unset http.proxy
git config --global --unset https.proxy
~~~

### 查询是否使用

~~~bash
git config --global http.proxy 
git config --global https.proxy 
~~~

