## [原创]使用git监控www文件并自动恢复

本文原创：**中国科学技术大学 张焕杰**

修改时间：2017.10.05

## 一、使用git监控www文件并自动恢复原理

使用多个分支来工作，其中：

* 自己更新的www文件在master分支

* 其他分支被用来记录异常更新

* 平时处于其他分支（以时间戳为分支名字）

* 设置post-commit 钩子，当有新提交时，做如下工作：  
    触发修改报警  
    执行`git checkout master`恢复文件  
    执行``git checkout -b `date '+%Y%m%d%H%M%S'` ``生成新的分支

* 接到报警后，之前的某个分支记录有修改的内容


## 二、设置过程

下文假定 /root/test/www下文件需要保护，设置过程为：

1. 初始化git库，并把www目录下文件全部加如git跟踪
````
cd /root/test
git init
git add www
git commit -m init
````

2. 生成post-commit 钩子文件
````
vi /root/test/.git/hooks/post-commit
````
写入以下内容
````
#!/bin/sh

cd /root/test

date >> change.log
echo "change!" >> change.log

git checkout master
git checkout -b `date '+%Y%m%d%H%M%S'`
````

3. 生成监控脚本程序

````
vi /root/test/jiankong.sh
````

写入以下内容：
````
#!/bin/sh
cd /root/test
while true; do
 git add www
 git commit -a -m `date '+%Y%m%d%H%M%S'`
 sleep 5
done
````

4. 把脚本改为可执行
````
chmod u+x .git/hooks/post-commit jiankong.sh
````

5. 编辑.gitignore，记录需要忽略的文件
````
vi /root/test/.gitignore
````
写入以下内容
````
.gitignore
jiankong.sh
change.log
````

## 三、修改www文件步骤

1. 停止上面的监控脚本jiankong.sh运行

2. 执行`cd /root/test; git checkout master`切换到master分支（记住我们的修改是在master分支）

2. 修改文件

3. 修改后执行`git status`，`git diff`等确认文件修改正确，如果有新增加的文件，可以执行`git add file_name`或`git add www`将文件加入git管理，然后执行`git commit -a -m "提交说明"`提交所有修改。

4. 注意一旦在master完成提交，post-commit自动执行，会切换到一个以修改时间戳为名字的分支。

5. 运行脚本jiankong.sh开始监控


## 四、实际案例

1. www目录只有一个文件index.html，内容是"index"，初始化步骤是：
````
[root@blackhole www]# cd /root/test
[root@blackhole test]# cat www/index.html
index
[root@blackhole test]# git init
Initialized empty Git repository in /root/test/.git/
[root@blackhole test]# git add www
[root@blackhole test]# git commit -m init
[master (root-commit) 96de862] init
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 www/index.html
````

2. 两个脚本文件内容如下：
````
[root@blackhole test]# ls -al  .git/hooks/post-commit jiankong.sh
-rwxr--r--. 1 root root 134 Oct  5 08:56 .git/hooks/post-commit
-rwxr--r--. 1 root root 153 Oct  5 08:57 jiankong.sh
[root@blackhole test]# cat  .git/hooks/post-commit
#!/bin/sh

cd /root/test

date >> change.log
echo "change!" >> change.log

git checkout master
git checkout -b `date '+%Y%m%d%H%M%S'`
[root@blackhole test]# cat jiankong.sh
#!/bin/sh
cd /root/test
while true; do
 git add www
 git commit -a -m `date '+%Y%m%d%H%M%S'`
 sleep 5
done

````
3. 自己修改文件演示，增加"my change"
````
[root@blackhole test]# cd /root/test; git checkout master
Already on 'master'
[root@blackhole test]# echo "my change" >> www/index.html
[root@blackhole test]# git commit -a -m "my change"
Already on 'master'
Switched to a new branch '20171005085844'
[20171005085844 a0e3e00] my change
 1 files changed, 1 insertions(+), 0 deletions(-)
[root@blackhole test]# cat www/index.html
index
my change
````
注意已经自动切换到分支 20171005085844

4. 运行监控脚本jiankong.sh，这时不会有任何变化

5. 模拟其他人修改

没有切换到master分支的修改都被认为是其他人修改。
````
[root@blackhole test]# echo "other change" >> www/index.html
[root@blackhole test]# cat www/index.html
index
my change
other change
````

6. 运行监控脚本jiankong.sh

other change会被记录到分支20171005085844，并被删除，切换到一个新的分支20171005090631
````
[root@blackhole test]# sh jiankong.sh
Switched to branch 'master'
Switched to a new branch '20171005090631'
[20171005090631 254a22d] 20171005090631
 1 files changed, 1 insertions(+), 0 deletions(-)
# On branch 20171005090631
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
nothing added to commit but untracked files present (use "git add" to track)
^C
[root@blackhole test]# git branch
  20171005085844
* 20171005090631
  master
[root@blackhole test]# cat www/index.html
index
my change
[root@blackhole test]# cat change.log
Thu Oct  5 08:58:44 CST 2017
change!
Thu Oct  5 09:06:31 CST 2017
change!

````

7. 模拟新增一个文件的恢复
````
[root@blackhole test]# echo "test2" > www/new.html
[root@blackhole test]# sh jiankong.sh
Switched to branch 'master'
Switched to a new branch '20171005091700'
[20171005091700 bd34034] 20171005091700
 1 files changed, 1 insertions(+), 0 deletions(-)
 create mode 100644 www/new.html
^C
[root@blackhole test]# ls -al www
total 12
drwxr-xr-x. 2 root root 4096 Oct  5 09:17 .
drwxr-xr-x. 4 root root 4096 Oct  5 09:16 ..
-rw-r--r--. 1 root root   16 Oct  5 09:06 index.html

````

## 五、优化
如果觉得傻傻的不停的git add; git commit太无聊，可以使用inotify-tools监测到文件变化再执行，把jiankong.sh修改为：
````
#!/bin/sh
cd /root/test
while true; do
 echo wait file system change..
 inotifywait -e modify,create -r /root/test
 git add www
 git commit -a -m `date '+%Y%m%d%H%M%S'`
done
````



***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)