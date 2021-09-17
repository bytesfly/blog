# Git使用汇总


git就不用多说了，有些配置或者不常用的命令容易忘，这里随手做个记录。

## git常用命令
```bash
# 只克隆最近1次commit
git clone --depth=1 repo_url.git

# example: 
# git clone -b release-1.9 --depth=1 git@github.com:apache/flink.git

# 添加远程仓库地址
git remote add itwild git@github.com:itwild/rewild.git

# 查看远程仓库地址
git remote -v

# 将本地当前分支推送到远程itwild仓库的master分支
git push itwild master

# 强推(需要关闭对分支的保护)，该操作需要谨慎
git push -f itwild master

# 修订上次的提交
git commit --amend

# 修订上次的提交时间
git commit --amend --date="1 minute ago"

# 指定commit时间
git commit --date="10 day ago" -m "Your commit message" 

# 调整前几次的提交
git rebase -i HEAD~2

# 从itwild远程仓库拉取最新的代码(仅仅是拉取代码，并没有参与本地分支merge或者rebase操作)
git fetch itwild

# 展示本地分支的提交历史
git log

# 展示远程分支itwild/master的提交历史
git log itwild/master

# 可取代merge操作，避免产生不必要的commit，破坏提交树的美观
# 找到两条分支的共同祖先，然后将共同祖先后不同的提交，回放一遍。
git rebase itwild/master

# 本地所有修改的，没有的提交的，都返回到原来的状态
git checkout . 
# 把所有没有提交的修改暂存到stash里面。可用git stash pop恢复
git stash

# 返回到某个commit，不保留修改
git reset --hard commit_id

# 返回到某个commit，保留修改
git reset --soft commit_id
```

从工作目录中删除所有没有tracked过的文件
```bash
git clean 参数
    -n  Don't actually remove anything, just show what would be done
    -f  --force  If the Git configuration variable clean.requireForce is not set to false, git clean will refuse to delete files or directories unless given -f, -n or -i. Git will refuse to delete directories with .git sub directory or file
           unless a second -f is given.
    -d  Remove untracked directories in addition to untracked files
```

cherry-pick某个提交
```bash
git cherry-pick [<options>] <commit-ish>...

常用options:
    --quit                退出当前的chery-pick序列
    --continue            继续当前的chery-pick序列
    --abort               取消当前的chery-pick序列，恢复当前分支
    -n, --no-commit       不自动提交
    -e, --edit            编辑提交信息
```

## git tag

```bash
# https://github.com/apache/flink/tags

# https://www.liaoxuefeng.com/wiki/896043488029600/902335479936480

# 本地打tag
git tag -a release-1.11.2 -m 'Apache Flink 1.11.2'

# 删除一个本地标签
git tag -d <tagname>

# 推送一个本地标签到远程仓库
git push itwild <tagname>

# 推送全部未推送过的本地标签
git push itwild --tags

# 删除一个远程标签
git push itwild :refs/tags/<tagname>
```

## git子模块(git submodule)
项目中可能会使依赖一些公共库或者其他team维护的其他项目，git submodule就可以很方便解决这样的场景。
使用子模块后，不必负责子模块的维护，只需要在必要的时候同步更新子模块即可。
```bash
# 递归克隆包含子模块的项目
git clone --recursive <repository>

# 添加子模块
git submodule add [repo]

git submodule init
git submodule update --init --recursive
git submodule update --remote --recursive

git submodule foreach 'git pull'
git submodule foreach 'git checkout -b featureA'
git submodule foreach 'git diff'
```

## git配置多个SSH-Key
参考：https://my.oschina.net/stefanzhlg/blog/529403

日常工作中会遇到公司有个gitlab，还有些自己的一些项目放在github上。这样就导致我们要配置不同的ssh-key对应不同的环境。具体操作如下：

1. 生成一个公司用的SSH-Key
```bash
ssh-keygen -t rsa -C "youremail@yourcompany.com" -f ~/.ssh/gitlab-rsa
```
在~/.ssh/目录会生成gitlab-rsa和gitlab-rsa.pub私钥和公钥。 我们将gitlab-rsa.pub中的内容粘帖到公司gitlab服务器的SSH-key的配置中。

2. 生成一个github用的SSH-Key
```bash
ssh-keygen -t rsa -C "youremail@your.com" -f ~/.ssh/github-rsa
```
在~/.ssh/目录会生成github-rsa和github-rsa.pub私钥和公钥。 我们将github-rsa.pub中的内容粘帖到github服务器的SSH-key的配置中。

3. 添加私钥
```bash
ssh-add ~/.ssh/gitlab-rsa
ssh-add ~/.ssh/github-rsa
```
如果执行ssh-add时提示"Could not open a connection to your authentication agent"，可以现执行命令：`ssh-agent bash`，然后再运行ssh-add命令。
```bash
# 可以通过 ssh-add -l 来确私钥列表
ssh-add -l
# 可以通过 ssh-add -D 来清空私钥列表
ssh-add -D
```

4. 修改配置文件  
   
在 ~/.ssh 目录下新建一个config文件
```bash
touch config
```
添加内容：
```bash
# gitlab
Host gitlab.com
    HostName gitlab.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitlab-rsa
# github
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github-rsa
```

5. 测试
```bash
# test your company gitlab
ssh -T git@yourcompany.com
# test github
ssh -T git@github.com
```

## git对某个项目单独设置用户名/邮箱
1. 进入项目.git文件夹，然后执行如下命令分别设置用户名和邮箱
```bash
git config user.name "bytesfly"
git config user.email "bytesfly@example.com"
#使用vim作为编辑器
git config --global core.editor "vim"
```
2. 然后可以查看生成的config文件
```bash
cat config
```

## git代理

取消代理：
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

设置代理：
```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

仅代理`GitHub`：
```bash
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080

# 取消代理
git config --global --unset http.https://github.com.proxy
```
补充：
如何自测本地代理是否有效(当然也可能端口有误)：
```bash
curl -x socks5h://127.0.0.1:1080 www.google.com
```
> In a proxy string, socks5h:// and socks4a:// mean that the hostname is resolved by the SOCKS server. socks5:// and socks4:// mean that the hostname is resolved locally

也就是说：
- `socks5`适合本地能够解析目标主机域名(比如`github.com`)但是访问速度慢,来提高下载速度
- `socks5h`用与本地不能解析目标主机域名(比如`google`),由代理服务器解析目标主机域名

```bash
export ALL_PROXY=socks5h://127.0.0.1:1080
```