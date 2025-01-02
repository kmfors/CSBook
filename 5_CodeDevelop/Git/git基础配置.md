git下载官网：[https://git-scm.com/](https://git-scm.com/)

参考资料：[https://zhuanlan.zhihu.com/p/607970211](https://zhuanlan.zhihu.com/p/607970211)

# 一、本地与托管的连接（SSH key）

主要是建立本地与代码托管平台之间的连接，可以供其后面代码的上传。

## 1、配置账户与邮箱

配置 Git 用户名与邮箱地址命令如下：

```shell
 git config --global user.name "Name"  
 git config --global user.email "email"
```

查看本地git的账户和邮箱，可使用上面命令去掉` --global` 即可。

## **2、生成密钥**

确保本地已经安装了ssh工具，推荐使用 ED25519密匙加密规则，以下也是只针对于ED25519进行介绍生成密匙。

```shell
# <comment> 一般填写邮箱
ssh-keygen -t ed25519 -C "<comment>"
```

- `ssh-keygen` 生成密钥命令
- `-t ed25519` 生成的密钥类型，这里是 ED25519 密钥
- `-C "email地址"` 允许添加一个标识信息，通常用来说明这个密钥是用于哪个账户或用途

```shell
# 密匙存放于默认的 SSH 密钥目录 ~/.ssh 中
id_ed25519           # 私钥文件，应该保持非常安全，不要分享给他人

id_ed25519.pub       # 公钥文件，用于 SSH 密钥身份验证的服务（如 GitHub、GitLab、远程服务器等）
```

## 3、将密钥添加代理

将产生的新 ssh key 添加到 ssh-agent 代理服务中，以便在 SSH 连接时不需要每次输入密码，注意是后缀为pub为公钥。

```shell
# 启动代理守护进程
eval $(ssh-agent -s) 

ssh-add ~/.ssh/id_ed25519
```

 若出现报错提示：`Could not open a connection to your authentication agent.` 说明并没有启动代码守护进程。

**注意：SSH 代理只在当前会话中有效，如果你注销或关闭终端，代理将被清除。你需要在每个新的会话中运行 `ssh-add` 来添加私钥到代理中，或者你可以使用 `ssh-agent` 来持久化代理，使其在用户登录时自动启动。**

## 4、添加 ssh公钥

这一步主要是将 SSH key 添加到你的 GitHub/ gitee / gitlab 账户中。
- 打开 **id_ed25519.pub ** 文件，将公钥复制后填写到账户中
- 本地执行 `ssh -T git@github.com` 或 `ssh -T git@gitlab.com` 命令对 ssh key 进行验证

执行成功结果：`You've successfully authenticated, but GITEE.COM does not provide shell access.`

> `ssh -T` 选项的意思为，不分配伪终端。 即无法使用 ssh 协议直接登录github，而是在github服务器上建立一个伪终端，并进行操作。 所以，这句提示并不是一个错误，而是github输出的一句提示语。可以在本地使用 ssh协议进行 git 相关操作，并提交到github，没有任何影响。

# 二、建立仓库

这里有建立仓库的3种情况
- 建立本地仓库，并推送到远程，远程不存在该仓库
- 建立本地仓库，并推送到远程，远程存在该仓库
- 拉取远程仓库，建立本地仓库

## 1、建立本仓推送，无远仓

**(1) 创建远程 Git 仓库**：首先创建一个远程的空仓库，才能为后续提供服务

**(2) 创建本地 Git 仓库**：进入新创建的目录，初始化本地 Git 仓库

```shell
git init
# 当前目录下生成 .git 目录，用于存储 Git 仓库的配置和历史信息
```

**(3) 添加和提交文件**：创建一个项目文件，将其添加并提交
```shell
 git add <文件名>  
 git commit -m "Initial commit"
```

**(4) 推送到远程仓库** ：先创不创建本地的master分支无所谓，最后本地与远程都自动创建master分支

```shell
git remote add origin <仓库git地址>  

git push -u origin master
```

> Tips：`-u` 或 `--set-upstream`： 这个选项用于建立本地分支和远程分支之间的关联。一旦建立关联，你可以使用 `git pull` 和 `git push` 命令，而不需要显式指定远程分支的名称，Git 将自动识别并操作与当前分支相关联的远程分支。

## 2、建立本仓推送，有远仓

参考上面的步骤，只不过有一些不同。推送文件至远程仓库，如果发现push报错。

原因：在两个git项目合并时，由于两个项目的目录结构不一致。这时推送和拉取都是不行的。解决办法：先合并本地分支，然后重新push。

```shell
# 将本地提交移动到拉取的远程分支的顶部，相当于先拉取后做自己的改动
git pull --rebase origin master
```

## 3、拉取远仓，建本仓

由于前面我们已经配置了SSH的连接，所以我们进行克隆的时候直接执行下列命令

```shell
git clone git@...
```
L

# 三、项目分支合并流程

目前已经有了一个master分支，现在主要做的开发是在dev分支下，你自己的一个本地分支做开发。

在进行合并时，首先就是将你自己的分支推送到远程下，因为自己的分支肯定是最新的没有人动的，要保持自己的分支是最新的稳定的。

切换到dev分支，pull一下dev，保持dev是最新的。切换到自己的分支下，合并dev分支。此时会有冲突，那就进行更改冲突，冲突解决后进行commit。然后再接着推送到自己的分支。再切换到dev分支下，合并自己的分支，应该是没有问题的。然后在进行推送dev分支。

![[20240607110049.png]]



问题：如果一个文件本地与远程都同步有，但是本地文件的删除并没有通过git命令删除，导致再想pull拉取远程的文件时，拉取不成功

问题分析：如果没猜错的话，当前的本地库commit落后于origin的commit，而且还改动了同一份文件，所以我们需要将本地库的 Head 指向origin的最新commit。

```shell
# 依次执行以下命令
git fetch --all  
git reset --hard origin/master  
git pull
```

# 四、分支问题

问题：已修改的文件**未添加**或**未提交**时，进行分支的切换。
后果：
- 若修改文件与切换的分支**无冲突**，Git 将允许切换分支，但会将你当前工作目录的修改带到新的分支中。
- 若修改文件与切换的分支**有冲突**，Git 将拒绝切换分支，并显示一个错误消息，提示你需要先处理冲突。

措施：在切换分支之前，**若确保与切换分支无冲突**，可以先将当前分支上的修改提交或者暂存来确保你的修改不会丢失。若不确定是否有冲突，可以**先将已修改文件添加到暂存区，然后使用 `git stash` 命令将你的修改保存在临时存储区中，再切换分支**。之后，你可以使用`git stash pop`来恢复你的修改。

注意：`git stash` 命令只能用于已经添加（git add）的文件使用，未添加或已提交状态无法使用。

---

# 以下未整理

问题：已修改文件如何撤销暂存、撤销提交

措施：

撤销添加：取消已经暂存（staged）但尚未提交的文件的修改，将它们恢复到未暂存的状态，同时保留工作目录中的修改。
```shell
git restore --staged <filename>	 #--staged: 这个选项告诉 Git 要取消暂存的是文件，而不是工作目录中的修改
```

撤销提交：撤销最近的一次提交，保留工作目录和暂存区的状态。
```shell
git reset HEAD~1
```


> 注意：
> 
> git restore  <filename>	#撤销暂存的同时，不保留工作目录的修改  
> git reset --hard HEAD~1   #撤销提交的同时，还原所有更改不保留