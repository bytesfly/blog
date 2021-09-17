# MacOS如何调整JD-GUI反编译工具字体大小

how to change the fontsize of JD-GUI in MacOS？  
MacOS如何调整JD-GUI反编译工具字体大小？

## 问题描述
JD-GUI是一款比较好用的反编译工具，不小心碰到什么东西把字体变得很小，代码都无法看清。在界面好像没找到调整字体大小的设置，双支、三指放大都不奏效。  
这里记录一下解决方案，让遇到同样问题的朋友少走弯路。

## 解决方法

Google搜索`jd gui font`排在第一位的链接就解决我的问题。`https://github.com/java-decompiler/jd-gui/issues/26`

```bash
# 我的是在这个目录，或者用find去搜jd-gui-1.4.1.jar或者jd-gui.cfg
cd /Applications/JD-GUI.app/Contents/Resources/Java

vim jd-gui.cfg

# 找到<ViewerPreferences.fontSize>12</ViewerPreferences.fontSize>，我设置为12，然后关掉JD-GUI，重新打开JD-GUI字体大小就正常了
```
随手记录，方便你我他。

## 2021.07.13更新

评论中有朋友说在GUI界面上也可以设置字体，如下：   
Help -> Preferences -> Appearance -> Font size

我在`Deepin Linux`系统 下载最新版本试了下，确实有设置字体的选项。另外，观察了一下，也在当前用户(`home`)目录下的`.config`目录中看到了`jd-gui.cfg`文件，如下：
```bash
➜  ~ realpath .config/jd-gui.cfg 
/home/bytesfly/.config/jd-gui.cfg
➜  ~ cat .config/jd-gui.cfg | grep font
                <ViewerPreferences.fontSize>12</ViewerPreferences.fontSize>
➜  ~ 

```

顺便也把最新版本下载链接放在这：  
[https://github.com/java-decompiler/jd-gui/releases](https://github.com/java-decompiler/jd-gui/releases)