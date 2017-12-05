## 参与我们的工作

如果您对git不熟悉，建议参考 [大家一起学git](https://github.com/bg6cq/learngit) 花2个小时熟悉git基本概念和操作。

如果您使用国内的gitee.com，请将下面的github替换为gitee即可。

由于github的存在，参与我们的工作非常简单，只需要3步：
1. fork代码，得到自己的拷贝
2. 在自己的拷贝里修改
3. 创建合并申请，等上游接受了合并，修改就更新到 [https://github.com/bg6cq/ITTS](https://github.com/bg6cq/ITTS)。

## 前期准备工作

1. 在github.com上申请账号（Sing up，其他略）

2. 在github.com上fork一份代码

    登录自己的github账号，访问[https://github.com/bg6cq/ITTS](https://github.com/bg6cq/ITTS)，单击右侧的Fork，现在https://github.com/xxx/ITTS 就是你自己的拷贝，在里面可以随意修改。

## 修改文档

可以在 github.com 在线修改文档，在 https://github.com/xxx/ITTS 自己的拷贝里可以创建文件、上传文件、修改文件，这里修改的都是自己拷贝中的。
直接在master中修改即可。

也可以将文件下载回自己的机器修改，请参见下面的 离线修改文件。

##  创建合并申请
创建一个pull request，等待上游接受合并

    访问 https://github.com/xxx/ITTS/pulls，“New pull request", 最右边方框选head fork:xxx/ITTS compare:master, 然后 "Create pull request"

## 与上游保持更新
上游更新后，访问 https://github.com/xxx/ITTS/ 自己的拷贝，会提示“This branch is ? commit behind bg6cq:master."。

最好的解决办法是使用命令行更新，最简单的办法是把自己的拷贝删除，重新fork。

单击提示右侧的"Compare", 单击 "bg6cq:master is up to date with all commits from xxx:master. Try switching the base for your comparison." 这里的 "switching the base"，单击"Create pull request", 在出来的页面单击"Merge pull request", "Confirm merge"

请参考[https://stackoverflow.com/questions/7244321/how-do-i-update-a-github-forked-repository](https://stackoverflow.com/questions/7244321/how-do-i-update-a-github-forked-repository)

## 离线修改文件

1. 安装git软件

    Linux可以安装git，windows建议到 [https://git-for-windows.github.io/](https://git-for-windows.github.io/) 下载安装
    设置好自己的邮件地址和姓名：
```
git config --global user.email "my@mail.org"
git config --global user.name "My Name"
```

2. 安装文本编辑器

    Linux可以使用自带的编辑器如vim等，windows系统下建议安装 [Notepad++](https://notepad-plus-plus.org/download/)

3. 在github.com上申请账号（略）

4. 在github.com上fork一份代码

    登录自己的github账号，访问[https://github.com/bg6cq/ITTS](https://github.com/bg6cq/ITTS)，单击右侧的Fork

5. 下载自己的代码

    在某个目录下，执行 `git clone https://github.com/xxx/ITTS.git` （其中xxx是自己的github账号）下载自己的代码，下载后的代码在目录ITTS下。

6. 设定上级源

```
git remote add bg6cq https://github.com/bg6cq/ITTS.git
```

7. 创建一个分支
```
cd ITTS
git checkout -b xxx
```

8. 在这个分支上修改

9. 提交到服务器
```
git add ??.md
git commit -m commit_message
git push origin xxx
```

10. 到服务器上可以看到效果，注意要选择branch xxx

11. 创建一个pull request

    访问 https://github.com/xxx/ITTS/pulls，“New pull request", 最右边方框选 compare:xxx, 然后 "Create pull request"

12. 等修改被接收后，执行以下命令
```
git pull bg6cq master
git push origin xxx
```


## 推荐阅读

1. [Pro Git 第二版（中文版）](https://www.kancloud.cn/kancloud/progit/70158)
