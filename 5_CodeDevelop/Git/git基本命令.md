



# 撤销本地 commit

一、方式 1

1、使用 `git log` 查询提交记录

2、使用 `git reset commitId` 回退到你想要的版本

（ps: commitId 就是 git log 里面显示的一长串字符，每次提交记录都有，你想要回退到哪个提交节点，就使用哪个 commitId）

二、方式 2

其中两种方式不清除本地提交和清除本地提交的方法

1、回退到上次提交并清除本地提交的代码：`git reset --hard HEAD^`
2、回退到上次提交不清除本地提交的代码：`git reset --soft HEAD~1` 

三、方式 3 (推荐)

问题：在 mster 分支写了半天，然后 git commit 提交了，才发现在 masrter 分支开发的。

解决：git reset HEAD~

git reset HEAD~
HEAD 代表：上一次提交

这样刚刚提交的就又回到本地的 local changes 列表中。nice
继续切换分支，重新提交


# 查看文件提交信息

**要查看指定文件的提交历史以及各次提交的提交者信息**，可以使用以下命令：

```shell
git log --follow -- <file_path>
```

解释命令选项：
- git log: 查看提交日志。
- --follow: 如果文件在历史上被重命名过，此选项能够跟踪文件的重命名历史，并显示文件的所有提交记录。
- -- <file_path>: 指定要查看历史记录的文件路径。

注意事项：
- 如果文件曾经被重命名或移动过，使用 --follow 选项可以帮助跟踪文件的整个历史。否则，只能显示文件在当前路径下的提交历史，而不会跟踪其移动或重命名前的历史。
- 如果文件的路径中包含空格或特殊字符，建议将文件路径用双引号括起来，例如：`git log --follow -- "path/to/file with spaces.cpp"`。
- 通过这些命令和选项，你可以详细查看指定文件的提交历史及其每次提交的作者信息。


# 查看分支基于哪条分支

git reflog show --all | grep "必要信息"

第一种情况：

```shell
git branch branch_name
git checkout branch_name
```

这种情况的日志是：

```shell
checkout: moving from father_branch to son_branch
branch: Create from father_branch
```

第二种情况：

```shell
git checkout -b son_branch
```

日志是：

```shell
branch: Create from HEAD
checkout: moving from father_branch to son_branch
```