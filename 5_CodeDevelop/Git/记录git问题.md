# 1. git 在 linux 命令行不能自动补全

(1) 执行命令：

```shell
curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash 
```

(2) 在 home 目录的`.bashrc`文件末尾增加以下内容：

```shell
if [ -f ~/.git-completion.bash ]; then 
. ~/.git-completion.bash 
fi 
```

(3) 重新加载`.bashrc`文件：

```shell
source ~/.bash_profile
```

(4) 请在 git 仓库目录测试，输入 `git com` 之后，然后按 `tab` 键即可出现自动补全。