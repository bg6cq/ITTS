## [原创] Claude Code常用资源

本文原创：**中国科学技术大学 张焕杰**

修改时间：2026.11.25

### What's agent doing

显示Cluade Code的状态
```
/plugin marketplace add tzafrir/whats-agent-doing
/plugin install whats-agent-doing@tzafrir
```

### Code Audit
Code Audit 是一个专业白盒代码安全审计技能

```bash
mkdir ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/3stoneBrother/code-audit
```
```
使用/code-audit deep 对这个项目进行安全审计，并形成完整的审计报告。
```
 
### bento html PPT

bento 是一个单一文件的用于演示的 html 文件，页面部分是json 格式，方便AI和人编辑。

源站点：https://github.com/nyblnet/bento

## 安装方法

Claude Code 中执行下面的命令会安装bento-slides
```bash
/plugin marketplace add https://github.com/nyblnet/bento
/plugin install bento-slides@bento
```

在Claude Code中，输入
```
从 *.md 或 *.docx 文件，使用 bento-slides 生成新的 html 演示文件
```

从已有的模版文件生成

从 https://bento.page/slides/ 下载样板，存为 sample.banto.html
在某个软件项目目录下，执行命令`claude`启动Cluade Code, 输入
```
从 *.md 或 *.docx 文件，参考 sample.bento.html 生成新的 html 文件
```



参考：




***
欢迎 [加入我们整理资料](https://github.com/bg6cq/ITTS)
