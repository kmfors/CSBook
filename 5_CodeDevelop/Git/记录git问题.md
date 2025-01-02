## git 在 linux 命令行不能自动补全

(1) 执行命令：

```shell
curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash 
```

(2) 在 home 目录的 `.bashrc` 文件末尾增加以下内容：

```shell
if [ -f ~/.git-completion.bash ]; then 
. ~/.git-completion.bash 
fi 
```

(3) 重新加载 `.bashrc` 文件：

```shell
source ~/.bash_profile
```

(4) 请在 git 仓库目录测试，输入 `git com` 之后，然后按 `tab` 键即可出现自动补全。



## 切换默认仓库推送

情景：我有两个远程仓库 origin 与 balck，现在 `git push` 时默认是向 black 仓库推送的，只有 `git push origin` 才向 origin 推送，该怎么设置才能让 `git push` 直接推送到 black 呢？

```shell
# 1、对特定分支进行追踪更改
git branch --set-upstream-to=origin/main main

# 2、查看分支追踪信息
git branch -vv

# 以下二者命令等价
git branch --set-upstream <local-branch> <remote-name>/<remote-branch>
git branch -u <local-branch> <remote-name>/<remote-branch>
```

