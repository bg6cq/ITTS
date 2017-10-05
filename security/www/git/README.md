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

* 接到报警后，可以从之前的某个分支看到修改的内容


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
git checkout master
git checkout -b `date '+%Y%m%d%H%M%S'`
while true; do
 git commit -a -m `date '+%Y%m%d%H%M%S'`
 sleep 5
done
````

4. 把脚本改为可执行
````
chmod u+x .git/hooks/post-commit jiankong.sh
````

## 三、修改www文件步骤

1. 停止上面的监控脚本jiankong.sh运行

2. 执行`cd /root/test; git checkout master`切换到master分支（记住我们的修改是在master分支）

2. 修改文件

3. 修改后执行`git status`，`git diff`等确认文件修改正确，如果有新增加的文件，可以执行`git add file_name`或`git add www`将文件加入git管理，然后执行`git commit -a -m "提交说明"`提交所有修改。

4. 运行脚本jiankong.sh开始监控


***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)