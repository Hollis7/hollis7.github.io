---
title: git学习
categories:
  - git
tags:
  - git
mathjax: true
---
<meta name="referrer" content="no-referrer"/>

## 初始化设置

配置⽤户名

~~~bash
git config --global user.name "hollis7"
~~~

配置邮箱

~~~bash
git config --global user.email "herryhollis@163.com"
~~~

查看配置信息

~~~bash
git config --global --list
~~~

> user.name=hollis
> user.email=herryhollis@163.com
> http.proxy=http://127.0.0.1:7890
> https.proxy=https://127.0.0.1:7890

## 创建仓库

创建⼀个新的本地仓库

~~~bash
git init
~~~

当前目录下指定一个仓库

~~~bash
git init my-repo
~~~

查看.git信息

~~~bash
ls -altr
ls -al
~~~

~~~bash
total 8
drwxr-xr-x 1 hdb22 197610 0 Nov 29 16:58 ../
drwxr-xr-x 1 hdb22 197610 0 Nov 29 16:58 ./
drwxr-xr-x 1 hdb22 197610 0 Nov 29 16:58 .git/
~~~

克隆⼀个远程仓库

~~~bash
git clone <url>
~~~

## 四个区域

**工作区(Working Directory)**
就是你在电脑里能实际看到的目录。

**暂存区（Stage/Index）**
暂存区也叫索引? ⽤来临时存放未提交的内容。 ⼀般在.git⽬录下的index中。

**本地仓库(Repository)**
Git在本地的版本库，仓库信息存储在.git这个隐藏目录中。

**远程仓库(Remote Repository)**
托管在远程服务器上的仓库。常用的有GitHub、GitLab、Gitee。

## 添加和提交

提交以.txt结尾的文件

~~~bash
git add *.txt
~~~

提交记录

~~~bash
$ git log
commit 965946e2d53df2d211bb4fa9ff357007815f9c3e (HEAD -> master)
Author: hollis <herryhollis@163.com>
Date:   Wed Nov 29 17:20:58 2023 +0800

    second commit

commit fccd50cb2e25533a639b4bc6ef4fa0d6c4225de6
Author: hollis <herryhollis@163.com>
Date:   Wed Nov 29 17:14:23 2023 +0800

    first commit

~~~

简洁日志

~~~bash
 git log --oneline
~~~



## git reset

<img src="C:\data\mysoftware\Typora\typoraPicture\image-20231129172540607.png" alt="image-20231129172540607" style="zoom: 25%;" />

### soft

查看版本id

~~~bash
git log --oneline
837739d (HEAD -> master) 3
e4b3f38 2
8ef07df 1
~~~

回退到指定版本id：`e4b3f38`

~~~bash
git reset --soft e4b3f38
~~~

~~~bash
$ git log --oneline
e4b3f38 (HEAD -> master) 2
8ef07df 1
~~~

查看暂存区的内容，`file3.txt`仍在

~~~bash
$ git ls-files
file1.txt
file2.txt
file3.txt
~~~

### hard

回退到上一个版本

~~~bash
git reset --hard HEAD^
~~~

> 工作区和暂存区的file3.txt均不存在

- [ ] 工作区
- [ ] 暂存区

### mixed

回退到上一个版本

~~~bash
git reset HEAD^
~~~

- [x] 工作区
- [ ] 暂存区

### 总结

~~~bash
git reset --soft id
git reset --hard HEAD^
git reset HEAD^
~~~

常用`git reset HEAD^`和`git reset --soft id`

:anger:慎用`git reset --hard HEAD^`

## 误操作

查看操作，回退到误操作之前：837739d

~~~bash
$ git reflog
e4b3f38 (HEAD -> master) HEAD@{0}: reset: moving to HEAD^
837739d HEAD@{1}: commit: 3
e4b3f38 (HEAD -> master) HEAD@{2}: commit: 2
8ef07df HEAD@{3}: commit (initial): 1
~~~

回退

~~~bash
 git reset 837739d
~~~

## 查看差异

1. 查看工作区和暂存区的差异

~~~bash
git diff
~~~

![image-20231129201440960](https://gitee.com/hollis7/pictures/raw/master/2023/11/29/93054_image-20231129201440960.png)

这里add一下，工作区和暂存区保持一致

~~~bash
 git add .
~~~

2. 比较工作区和版本库之间的差异

~~~bash
git diff head
~~~

![image-20231129202024642](https://gitee.com/hollis7/pictures/raw/master/2023/11/29/67476_image-20231129202024642.png)

3. 比较暂存区和版本库之间的差异

~~~bash
git diff --cached
~~~

![image-20231129202403474](https://gitee.com/hollis7/pictures/raw/master/2023/11/29/55545_image-20231129202403474.png)

4. 比较两个版本内容

~~~bash
 git diff 837739d e4b3f38
 git diff head 837739d
~~~

![image-20231129203139207](https://gitee.com/hollis7/pictures/raw/master/2023/11/29/13924_image-20231129203139207.png)

5. 最新两个版本之间

~~~bash
git diff head~ head
~~~

`head~`表示最新版本的上一版本，`head^`同理

`head~2`表示head之前的2个版本

6. 查看指定文件版本之间的差异

~~~bash
git diff head~ head file3.txt
~~~

## git rm删除文件

~~~bash
rm file1.txt
~~~

暂存区中的内容没有被删掉

~~~bash
$ git ls-files
file1.txt
file2.txt
file3.txt
~~~

需要add 和commit

### git rm

直接删除工作区和暂存区中的文件

~~~bash
git rm file2.txt
~~~

> 最后还是要提交，否则删除的文件还是在版本库中

### 附加

递归删除某个目录下的所有子目录和文件

~~~bash
git rm -r *
~~~

## 学习.gitigonore

~~~bash
echo "some log" > access.log
echo "other log " > other.log
~~~

将access.log放入`.gitigonore`文件

~~~bash
echo access.log > .gitignore
~~~

![image-20231129205548067](https://gitee.com/hollis7/pictures/raw/master/2023/11/29/13517_image-20231129205548067.png)

~~~bash
git add .
git commit -m "ignore and other files"
~~~

### 忽略所有日志文件

```bash
vim .gitignore
*.log
```

输入：`*.log`然后`wq`保存

### 测试效果

~~~bash
 echo hello > hello.log
 git status
~~~

![image-20231129210211915](https://gitee.com/hollis7/pictures/raw/master/2023/11/29/38701_image-20231129210211915.png)

~~~bash
 git commit -am "test ignore log"
 git ls-files
~~~

没有 `hello.log`

### 测试已经提交的other.log

追加

~~~bash
echo " modified" >> other.log
~~~

~~~bash
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   other.log
~~~

`.gitigonore`对已经添加的不能起作用



### 删除暂存区的other.log

~~~bash
git rm --cached other.log
~~~

- [ ] 暂存区
- [x] 本地

~~~bash
git commit -am "delete other.log"
~~~

> `git commit -am "delete other.log"` 这个命令的作用是将所有已跟踪的文件的修改提交到版本库，并且使用 "delete other.log" 作为提交的消息。

> 需要注意的是，这种方式只适用于已经被 Git 跟踪的文件。对于新添加的文件，还是需要使用 `git add` 将其添加到暂存区，然后再使用 `git commit -m "message"` 提交。

### 忽略文件夹

~~~bash
echo "hello" > temp/hello.txt
~~~

~~~bash
 vim .gitignore
 //追加temp/
 temp/
~~~

~~~bash
 git add .
 git commit -m "test ignore fold"
 git ls-files
 
 $ git ls-files
.gitignore
file3.txt
~~~

## 关联本地仓库和远程仓库

```bash
git remote add origin git@github.com:Hollis7/git_testrepo.git
git branch -M main
git push -u origin main
```

在GitHub上添加readme文件

本地仓库拉取远程仓库的修改

~~~bash
git pull
~~~

~~~bash
git remote add <remote-name> <remote-url>
//添加远程仓库
git remote -v
//查看远程仓库?
~~~

## ⽂件状态

- main/master 默认主分⽀
- Origin 默认远程仓库
- HEAD 指向当前分⽀的指针
- HEAD^ 上⼀个版本
- HEAD~ 上四个版本

## vscode中使用git

在git中打开vscode

~~~bash
code .
~~~

## 分支简介和基本操作

### 查看分支

~~~bash
git branch
~~~

### 创建新的分支

~~~bash
 git branch dev
~~~

查看分支，默认分支仍然为`master`

~~~bash
 git branch
  dev
* master
~~~

### 切换分支

~~~bash
git checkout dev（不建议，存在歧义）
git switch master（建议）
~~~

~~~bash
$ git checkout dev
Switched to branch 'dev'
~~~

~~~bash
$ git switch master
Switched to branch 'master'
~~~

## merge合并

在`dev`分支下创建`dev1.txt dev2.txt`，并提交，在`master`分支下查看，并没有`dev1.txt dev2.txt`

在创建main4.txt和main5.txt看到明显的分叉

:memo:合并分支,将`dev`分支合并到`master`（当前）分支

~~~bash
hdb22@hollis7 MINGW64 /c/individualproject/git-learn/branch-demo (master)
$ git merge dev
~~~

查看分支图

~~~bash
 git log --graph --oneline --decorate --all
~~~

~~~bash
$ git log --graph --oneline --decorate --all
*   554c860 (HEAD -> master) Merge branch 'dev'
|\
| * 01f908c (dev) dev:2
| * 0db7fa1 dev:1
* | 7e18db1 main:5
* | 5d73401 main:4
|/
* 185b91e main:3
* 823dd12 main:2
* 262a6df main:1
~~~

### 删除分支

合并后分支仍然存在，可以删除

~~~bash
 git branch -d dev
~~~

`-d`表示删除已经合并的分支

`-D`表示强制删除

## 合并分支解决冲突

~~~bash
git branch feat
git swich feat
vim main1.txt
~~~

在`feat`分支下提交修改的main1.txt

~~~bash
git commit -a -m "feat:1"
~~~

切回`master`分支，`main1.txt`的内容不变

~~~bash
 git switch master
 cat main1.txt
~~~

> main1

修改`main1.txt`并保存

~~~bash
vim main1.txt
git commit -am "main:6"
~~~

:memo:**直接合并出现冲突**

~~~bash
$ git merge feat
Auto-merging main1.txt
CONFLICT (content): Merge conflict in main1.txt
Automatic merge failed;fix conflicts and then commit the result.
~~~

使用`git diff`查看冲突内容

![image-20231201153625765](https://gitee.com/hollis7/pictures/raw/master/2023/12/01/66112_image-20231201153625765.png)

**手动修改main.txt文件，然后提交**

~~~bash
vim main1.txt
git add .
git commit -m "merge conflict"
~~~

提交之前中断合并可以使用

~~~bash
git merge --absort
~~~

## rebase

1. 为了便于演示，先删除feat分支

~~~bash
 git branch -d feat
~~~

2. 恢复dev分支

~~~bash
 git checkout -b dev 01f908c
~~~

`01f908c`是提交dev2.txt的id

3. 切回master，并reset到状态提交`main.5`的状态

~~~bash
git swtich master
git reset --hard 7e18db1
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/12/01/64965_image-20231201155844275.png" alt="image-20231201155844275" style="zoom:67%;" />

<img src="https://gitee.com/hollis7/pictures/raw/master/2024/01/16/10240_image-20231201160041852.png" alt="image-20231201160041852" style="zoom: 67%;" />

4. **回退目录，copy两次branch-demo分别演示**

~~~bash
cp -rf branch-demo rebase1
cp -rf branch-demo rebase2
~~~

5. rebase1文件夹下，切换到`dev`分支进行`rebase`，dev变基到master

~~~bash
 git switch dev
 git rebase master
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/12/01/84395_image-20231201160653932.png" alt="image-20231201160653932" style="zoom: 67%;" />

6. rebase2文件夹下，切换到`master`分支进行`rebase`，master变基到dev

~~~bash
 git rebase dev
~~~

<img src="https://gitee.com/hollis7/pictures/raw/master/2023/12/01/70958_image-20231201161355387.png" alt="image-20231201161355387" style="zoom:67%;" />

### 小结

只是想要merge，不关系历史采用rebase（个人开发，清晰明了）

共享分支采用merge，不然会给别人带来困扰



## alias

定义命令别名

~~~bash
 alias graph="git log --oneline --graph --decorate --all"
~~~

输入`graph`直接查看图像和提交记录

~~~bash
$ graph
*   9d6c599 (master) merge conflict
|\
| * 55416ce feat:1
* | 5b8f4ef main:6
|/
*   554c860 Merge branch 'dev'
|\
| * 01f908c (HEAD -> dev) dev:2
| * 0db7fa1 dev:1
* | 7e18db1 main:5
* | 5d73401 main:4
|/
* 185b91e main:3
* 823dd12 main:2
* 262a6df main:1
~~~

