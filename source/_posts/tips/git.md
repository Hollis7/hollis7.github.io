---
title: git小技巧
categories:
  - git
tags:
  - git
mathjax: true
---
<meta name="referrer" content="no-referrer"/>

## git设置代理

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

### idea中配置代理（可选？）

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/11/27/42136_image-20231127202015854.png" alt="image-20231127202015854" style="zoom: 67%;" />

7890是clash的端口