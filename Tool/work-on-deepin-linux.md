# Linux工作环境搭建——deepin系统的使用

上大学的时候就在自己的笔记本上安装过深度操作系统(deepin)，当时好像是15.x的版本。毕业后第一家公司是全Mac办公，因在学校期间有过完全Linux环境下的开发体验，上手Mac非常快、非常爽。前段时间换了工作，当前公司用的是台式机。于是，入职当天重装了deepin系统，也就有了此篇博客。随手记录，方便你我他。持续更新~~

![](https://img2020.cnblogs.com/blog/1546632/202105/1546632-20210516142452754-1442240469.png)

deepin最新版本下载  
[https://www.deepin.org/zh/download/](https://www.deepin.org/zh/download/)


如何安装deepin  
[https://www.deepin.org/zh/installation/](https://www.deepin.org/zh/installation/)


当前版本deepin20.2，开箱就内置了很多实用的软件。但作为软件开发人员，还需要安装一些开发中常用的工具与软件。

## 常用软件安装
```bash
sudo apt-get install git -y

sudo apt-get install curl -y

sudo apt-get install zsh -y

sudo apt-get install xsel -y

sudo apt-get install htop -y
```
Oh My Zsh安装：  
[https://ohmyz.sh/](https://ohmyz.sh/)

Chrome浏览器:  
https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb


Chrome浏览器常用插件

- Adblock Plus
- Tampermonkey
- JSON Formatter
- Sourcegraph
- Octotree
- GitCodeTree
- XPath Helper
- yuque-helper
- Google翻译
- Proxy SwitchyOmega



搜狗输入法：  
[https://pinyin.sogou.com/linux/](https://pinyin.sogou.com/linux/)

百度输入法：  
[http://srf.baidu.com/site/guanwang_linux/index.html](http://srf.baidu.com/site/guanwang_linux/index.html)

JDK：  
[https://repo.huaweicloud.com/java/jdk/8u202-b08/](https://repo.huaweicloud.com/java/jdk/8u202-b08/)  
[https://enos.itcollege.ee/~jpoial/allalaadimised/jdk8/](https://enos.itcollege.ee/~jpoial/allalaadimised/jdk8/)

Python开发环境搭建：  
[https://www.cnblogs.com/bytesfly/p/python-environment.html](https://www.cnblogs.com/bytesfly/p/python-environment.html)

Java反编译图形化工具：  
[http://java-decompiler.github.io/](http://java-decompiler.github.io/)

JetBrains全家桶：  
[https://www.jetbrains.com/zh-cn/products/](https://www.jetbrains.com/zh-cn/products/)


IntelliJ IDEA：  
[https://www.jetbrains.com/zh-cn/idea/download/other.html](https://www.jetbrains.com/zh-cn/idea/download/other.html)


PyCharm：  
[https://www.jetbrains.com/pycharm/download/other.html](https://www.jetbrains.com/pycharm/download/other.html)


DataGrip：  
[https://www.jetbrains.com/zh-cn/datagrip/download/other.html](https://www.jetbrains.com/zh-cn/datagrip/download/other.html)


VSCode:  
[https://code.visualstudio.com/Download](https://code.visualstudio.com/Download)


百度网盘客户端(官方已有Linux版)：  
[https://pan.baidu.com/download/](https://pan.baidu.com/download/)

Free Download Manager(也可以从deepin的应用商店直接安装):  
[https://www.freedownloadmanager.org/zh/](https://www.freedownloadmanager.org/zh/)


docker安装  
[https://wiki.deepin.org/wiki/Docker](https://wiki.deepin.org/wiki/Docker)  
另外附上别人已经整理好的安装脚本(实测没毛病, 强烈推荐)  
[https://gist.github.com/madkoding/3f9b02c431de5d748dfde6957b8b85ff](https://gist.github.com/madkoding/3f9b02c431de5d748dfde6957b8b85ff)



命令导入OpenVPN文件：  
[https://github.com/linuxdeepin/dde-control-center/issues/43](https://github.com/linuxdeepin/dde-control-center/issues/43)

[https://bbs.deepin.org/post/205870](https://bbs.deepin.org/post/205870)
```bash
sudo nmcli connection import type openvpn file your-own-openvpn-profile-config-file.ovpn
```


此外在deepin的应用商店可方便的安装很多常用软件，比如：  
Typora(markdown编辑器)、微信、QQ、WPS、迅雷、Postman、Wireshark、Flameshot(好用的截图工具)、网易云音乐等等

Typora主题：[https://theme.typora.io/theme/Drake/](https://theme.typora.io/theme/Drake/)  
字体：[https://www.jetbrains.com/zh-cn/lp/mono/](https://www.jetbrains.com/zh-cn/lp/mono/)


星火应用商店——致力于丰富Linux生态，取`星星之火，可以燎原`之意(有一些民间wine打包的应用):  
[https://www.spark-app.store/](https://www.spark-app.store/)


其他：  
[https://github.com/shadowsocksrr/electron-ssr](https://github.com/shadowsocksrr/electron-ssr)

[https://github.com/Qv2ray/Qv2ray](https://github.com/Qv2ray/Qv2ray)

[https://qv2ray.net/lang/zh/](https://qv2ray.net/lang/zh/)

## Command汇总

### 实用

命令行操作剪贴板
```bash
# 安装xsel
sudo apt-get install xsel

# 拷贝到剪贴板
cat file.txt | xsel -b

# 从剪贴板粘贴
xsel -b >> example.txt
```


字体查看
```sh
# 查看系统字体
fc-list

# 查看系统中已经安装的中文字体
fc-list :lang=zh

fc-list -q 'Noto Serif CJK SC'
fc-list -q 'WenQuanYi Micro Hei'
```

## 常见问题汇总


下面是使用deepin过程中遇到的常见问题汇总。持续更新~~


### 快捷键冲突


参考：[https://www.jianshu.com/p/4bbae666abff](https://www.jianshu.com/p/4bbae666abff)


IDEA中有好几个常用的快捷键被deepin系统占用了，非常难受，我是不愿意修改IDEA默认快捷键的(通用的多好哇)，所以尝试去修改deepin系统默认快捷键。
```bash
# 查看哪些快捷键被占用了，记得用grep过滤
gsettings list-recursively

# 取消Ctrl+Alt+U
gsettings set com.deepin.dde.keybinding.system  translation '[]'
```
修改被系统占用的快捷键`Ctrl+Alt+B`，这样IDEA中就能happy地使用了。
![](http://img2020.cnblogs.com/blog/1546632/202103/1546632-20210313093911720-306157024.png)

### dpkg: 处理软件包 xxx (--configure)时出错

使用`apt-get`安装某软件包(比如`xxx`)时失败可能会导致安装其他软件都失败，报错信息大致如下：
> dpkg: 处理软件包 xxx (--configure)时出错：
已安装 xxx 软件包 post-installation 脚本 子进程返回错误状态 1
在处理时有错误发生：
xxx
E: Sub-process /usr/bin/dpkg returned an error code (1)

可以用如下方法尝试解决(比如发生错误的软件包是`xxx`)：
```bash
# 新建临时目录用作备份
mkdir /tmp/xxx

# 查看xxx软件包信息文件
ls -l /var/lib/dpkg/info/xxx.*

# 把xxx软件包信息移到临时目录
sudo mv /var/lib/dpkg/info/xxx.* /tmp/xxx

# 下面这一步可以不要
sudo apt autoremove xxx
```
然后再用`apt-get`安装其他软件。

## 写在后面


当前只是记录了少许痕迹，随着后续对deepin的深度使用，更多使用建议与问题汇总将记录于此。也欢迎朋友在评论区留言，分享你的常用软件与经验总结！
