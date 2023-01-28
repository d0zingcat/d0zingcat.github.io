---
title: "1password SSH Key 管理实践"
description: 
date: 2022-10-15T14:12:20+08:00
slug: 1password-ssh-key-in-practice
categories: ['Tutorial']
tags: ['1password', 'ssh', 'git', 'Github']
draft: false
---

# 起因

在上一篇文章[^1]中有介绍过关于使用1Password做Github的commit Sign，当具体尝试之后我发现这种做法比 GPG 简单太多，而且很好管理，GPG单独维护一套很难做脚本的自动化，如果真的换机了之后应该还是需要重新找资料重新配置一遍，而且备份和还原方案也比较复杂，并不是我理想中的无痛的迁移方案，因此如果是需要做Github的commit 签名还是推荐这种方式，具体的方法可以参考我在上篇文章中推荐的资料。

# 步骤

我能想到的仅有的两个缺点一个是需要配置SSH的IdentityAgent, 而这个地址其实可以通过软连接的方式直接引用到`~/.1password/agent.sock`（这可以通过迁移的脚本来自动处理），以及（目前我掌握的信息来看）并没有一个方案可以在ssh config中指定连接时要使用的1Password中具体的某个SSH Key，因此还是在实际连接中程序会自动尝试所有存储在1Password中的key （虽然一般key的数量不会很多而且处理起来也很快，但总归是很不好的实现）。虽然可以通过下载Public Key到本地在ssh config中指定`IdentitiesOnly` [^5]和`IdentityFile`来控制，但并不是所有的程序都支持（例如我工作中依赖的Datagrip就不支持）[^6]。

根据1Password官方的教程[^2]可以把大部分的配置都完成，本文不会在具体介绍。这个模块下的所有的文章都值得读，但重点可以关注`Get Started`, `Manage SSH Keys`, `SSH agent`三个部分。另外官方也有写博客[^3]来的介绍这个功能，以及社区中别人的写的教程也很值得读[^4]。

# 排错

需要提示的是当我尝试替换掉我所有的SSH Key成1Password中配置的Key时，我才发现原先我并没有成功配置。
有两个关键点，一个是`~/.ssh`中还是存在`id_rsa`和`id_rsa.pub`两个文件，因此大部分server的访问还是通过`.ssh`目录下的密钥文件来访问的，换言之压根没有读取到1Password中的key；一个是我设置ssh-agent时是通过软链的方式连接到了`~/.1password`中，但是我并没有使用全路径创建的软连接(命令是`mkdir -p ~/.1password && ln -s ~/Library/Group\ Containers/2BUA8C4S2C.com.1password/t/agent.sock ~/.1password/agent.sock`)，所以看起来软连接是失效的。但实际上做Git Commit签名的时候还是正常的，这个问题我暂时没有明白。但通过shell查询这个目录下时，可以发现软链接是红色也就是失效的。于是通过命令`mkdir -p ~/.1password && ln -sf /Users/{username}/Library/Group\ Containers/2BUA8C4S2C.com.1password/t/agent.sock /Users/{username}/.1password/agent.sock`重新创建软链接之后即可发现软链接已经正常了。然后把`~/.ssh`中key都删除之后重新连接server即可发现能够正常触发1Password的验证了。

另外，在实际操作的过程中有发现Datagrip[^8]会无法正常使用SSH Tunnel来查询SQL。之前可以是直接读取的`~/.ssh/id_rsa`作为私钥来认证，而这次我把本地的私钥和公钥都删除了，因此原先的SSH tunnel配置就失效了（`Tools-SSH Configurations`中的Authentication type 从Key pair，也就是本地文件方式改成OpenSSH config and authentication agent）。但并没有生效，而`~/.ssh/config` 中也配置了下面的配置，理论上可以控制所有的SSH 连接。
```
Host *
    IdentityAgent "~/.1password/agent.sock"
```
具体到报错提示：
```
Authentication using ssh-agent

Unable to connect to ssh-agent: "~/Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock" path to socket is incorrect.   

Using ssh-agent during SSH key-based authentication is disabled.

```
1Password中的Team member给了解决方案[^7]，也很简单，在`Settings-Advanced Settings-SSH-Configuration files parser`从默认的Legacy改成OpenSSH 即可。

# 更进一步 2023.01.28

最近拿到了新电脑，一台公司用（且称呼为air），一台私人用（且称呼为pro）。但比较特殊的是公司要求使用公司邮箱注册Github账号，那么密钥的管理上面就会存在一些差异，因为**Github不允许在两个账号下添加同一个公钥**。不过有意思的是Github允许同一个公钥同时添加到`Authentication Keys` 和 `Signing keys`，那么做法就比较清晰了：

1. 废弃我之前的密钥对，新生成两对密钥（密钥类型使用更加先进的ed25519），分别称呼为`Personal_cipher`和`Work_cipher`，存储在1Password中并下载公钥存储到本地`~/.ssh/`中，分别命名为`personal_git.pub`和`work_git.pub`
2. 准备好两个Github账号，分别称呼为 `Personal_github`和`Work_github`
3. 把`Work_cipher`的公钥添加到`Work_github` `Settings-SSH and GPG keys`中，同时添加为`Authentication Keys` 和 `Signing keys`
4. 把`Personal_cipher`的公钥添加到`Personal_github` `Settings-SSH and GPG keys`中，同时添加为`Authentication Keys` 和 `Signing keys`
5. 如下配置`~/.ssh/config`
6. 对个人/公司仓库，配置不同的remote地址[^9]，以及配置对应的用户名/邮箱。

*ssh config*

```ini
Host *
    IdentityAgent "~/.1password/agent.sock"
    ServerAliveInterval 60
    TCPKeepAlive yes

# 可不配置，但建议配置，好处是可以在大陆使用时更加省心
# 例如当VPN供应商为了解决爆破滥用的问题，在防火墙策略上禁用了22出口，
# 那么在使用VPN的时候原先直接使用ssh的22端口进行代码的拉取推送等操作
# 会被拦截，提示密钥交换失败连接中断等奇奇怪怪的问题
Host github.com
    Hostname ssh.github.com
    Port 443
    User git

# Personal GitHub
Host personalgit
    Hostname ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/personal_git.pub
    IdentitiesOnly yes

# Work GitHub
Host workgit
    Hostname ssh.github.com
    Port 443
    User git
    IdentityFile ~/.ssh/work_git.pub
    IdentitiesOnly yes
```

当然，全局也可以根据不同的电脑配置不同的默认账户/签名的公钥，对于原先没有更改过remote的仓库就还是可以按照默认的名称、签名密钥去流转，这个可以参考具体git的命令或是写好的gitconfig文件[^a]。

下面是一个简单的shell函数，可以酌情添加在`~/.zshrc`中进行使用
```shell
function replace_remote() {
    if (( $# != 1 ));
    then
        echo 'Invalid parameter!'
    else
        url=$(git remote -v | head -n 1  | cut -d $'\t' -f 2 | cut -d ' ' -f 1)
        suffix=$(echo $url | cut -d ':' -f 2)
        case $1 in
            work)
                new_url=workgit:$suffix
                git remote set-url origin $new_url
                ;;
            personal)
                new_url=personalgit:$suffix
                git remote set-url origin $new_url
                ;;
            *)
                echo 'Invalid parameter'
                ;;
        esac
    fi
}
```
使用方法则是`replace_remote work|personal`，而更改git用户名和邮箱则可以通过`git config user.name "billgates" && git config user.email "jeff@google.com"`来实现，如果要设置全局的`config`后面跟` --global `即可[^b]。


[^1]: [如何给Github commit启用GPG Sign签名](https://blog.d0zingcat.dev/p/how-to-enbale-gpg-sign-for-github/#%E8%A1%A5%E5%85%85)
[^2]: [1Password for SSH & Git](https://developer.1password.com/docs/ssh)
[^3]: [SSH and Git, meet 1Password 🥰](https://blog.1password.com/1password-ssh-agent/)
[^4]: [How to set up the 1Password SSH agent for secure SSH connections](https://tinkerwell.app/blog/how-to-set-up-the-1password-ssh-agent-for-secure-ssh-connections) | [1Password + SSH = Terminal Bliss](https://crossedwires.substack.com/p/1password-ssh-terminal-bliss) | [1Password for SSHを試してみる](https://qiita.com/ohakutsu/items/083dcd86d4a0b887461d) | [Github等で利用するSSH keyの作成及び管理を1Passwordで行う](https://dev.classmethod.jp/articles/1password-git-ssh/)
[^5]: [IdentitiesOnly in ssh_config](https://www.georglutz.de/blog/2014/03/30/identitiesonly-in-ssh_config/)
[^6]: [SSH client compatibility](https://developer.1password.com/docs/ssh/agent/compatibility/)
[^7]: [Request: identity agent compatibility with data grip](https://1password.community/discussion/127788/request-identity-agent-compatibility-with-data-grip)
[^8]: [SSH Configurations](https://www.jetbrains.com/help/datagrip/settings-tools-ssh-configurations.html)
[^9]: [Use multiple GitHub accounts](https://developer.1password.com/docs/ssh/agent/advanced/#use-multiple-github-accounts)
[^a]: [~/.gitconfig](https://github.com/d0zingcat/dotfiles/blob/main/git/config)
[^b]: [How can I change the author name / email of a commit?](https://www.git-tower.com/learn/git/faq/change-author-name-email)
