# SSH 密钥连接配置

主要是建立本地与代码托管平台之间的连接，可以供其后面代码的上传。

git 下载官网：https://git-scm.com

参考资料：https://zhuanlan.zhihu.com/p/607970211

## 配置全局用户信息

```shell
 # 配置全局用户信息（若只为特定仓库配置用户信息，去掉 --global 即可）
 git config --global user.name "Name"  
 git config --global user.email "email"
```

补充命令：

```shell
# 查看全局配置（查看当前仓库的配置，去掉 --global 即可）
git config --global --list

# 或直接查看特定项
git config --global user.name
git config --global user.email
```



## 生成 SSH 密钥 (ED25519)

推荐使用 ED25519 密匙加密规则，以下也是只针对于 ED25519 进行介绍生成密匙。

```shell
ssh-keygen -t ed25519 -C "your.email@example.com"
```

- `ssh-keygen` 生成密钥命令
- `-t ed25519` 指定密钥类型，使用 ED25519 密钥
- `-C "email地址"` 添加标识信息，用来说明这个密钥的备注信息

执行完上面的命令，一路回车，会在 `~/.ssh` 中，生成两个密匙文件：

- **id_ed25519**：私钥文件，应该保持非常安全，不可分享
- **id_ed25519.pub**：公钥文件，用于 SSH 密钥身份验证的服务（如 GitHub、GitLab、远程服务器等）



## 将私钥添加到 SSH 代理

问题 1：`ssh-agent` 有什么用，若私钥不添加到该 ssh 代理服务中会怎样？

- 主要是密码记忆功能，只添加一次私钥，后续的 ssh 操作或 git 命令不用每次进行私钥添加

问题 2：为什么要将 `ssh-agent` 设置为守护进程？

- SSH 代理的启用，只在当前终端会话中有效，如果注销或关闭终端，代理将被清除
- SSH 代理启动被设置为守护进程后，SSH 代理独立于当前终端运行，不会随终端关闭而立即退出

```shell
# 启动代理，设为守护进程
eval "$(ssh-agent -s)" 

# 添加私钥到代理中
ssh-add ~/.ssh/id_ed25519
```

若出现报错提示：`Could not open a connection to your authentication agent.` 说明并没有启动代码守护进程。



## 在托管平台添加公钥

- 复制 `~/.ssh/id_ed25519.pub` 内容，添加到 GitHub/GitLab 等平台的 SSH Keys 设置中
- 验证连接：`ssh -T git@github.com`

执行成功结果：`You've successfully authenticated, but GITEE.COM does not provide shell access.`

> [!NOTE]
>
> `ssh -T` 选项的意思是 "不分配伪终端"。GitHub 出于安全考虑，**不提供任何形式的 shell 访问权限**，无论是交互式登录还是命令执行。当使用 `ssh -T git@github.com` 测试时，GitHub 会验证你的 SSH 密钥，然后立即关闭连接，同时显示认证成功的提示信息。



## ssh 免密登录

将本地公钥匙上传至远程服务器命令：

```shell
ssh-copy-id -p 22 root@xxxx.com
```

若上传命令不可用，可手动上传公钥：

```shell
cat ~/.ssh/id_ed25519.pub
echo "your_public_key_contents" >> ~/.ssh/authorized_keys
```

打开VSCode，编辑remote配置文件

```
Host myserver
    HostName xxxx.com
    User root
    IdentityFile ~/.ssh/id_ed25519
```

