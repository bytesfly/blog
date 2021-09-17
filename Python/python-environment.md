# Python开发环境搭建

好像任何一门编程语言都绕不过开发环境的搭建。比如说`Java`，初学者可能还没明白什么是`JDK`，但一般都会按照前辈们的步骤，先下载`JDK`，然后添加环境变量`JAVA_HOME`，再把`JAVA_HOME`的`bin`目录加入到环境变量`PATH`中，有过实际项目开发经验的朋友大多数紧接着就会安装依赖管理、项目构建工具，比如说国内常用的`Maven`等。`Golang`、`Scala`语言的开发环境搭建也都类似。至于用什么编辑器或者`IDE`来写代码，取决于语言特性或个人习惯，比如当前主流的`vim`、`VS Code`、`JetBrains`全家桶、`Eclipse`等等。


使用`Python`语言的初学者建议直接安装`Anaconda`或者轻量级的`Miniconda`。

## 为什么需要Anaconda

`Anaconda`官网：`https://www.anaconda.com/`

> Anaconda 是一个用于科学计算的 Python 发行版，支持 Linux, Mac, Windows, 包含了众多流行的科学计算、数据分析的 Python 包。

介绍`Anaconda`之前必须要简单提一下`Conda`。官网：`https://docs.conda.io/en/latest/`

> Conda is an open source package management system and environment management system that runs on Windows, macOS and Linux. Conda quickly installs, runs and updates packages and their dependencies. Conda easily creates, saves, loads and switches between environments on your local computer. It was created for Python programs, but it can package and distribute software for any language.

`Conda`是`包管理器`。使用Python进行数据分析时，你会用到很多第三方的包，`Conda`可以很好地帮助你在计算机上安装和管理这些包，包括安装、卸载和更新包。

`Conda`也是`环境管理器`。比如你在A项目中用了`Python2`，而B项目需要使用`Python3`，同时安装两个`Python`版本可能会造成许多混乱和错误。这时候，`Conda`就可以帮助你为不同的项目建立不同的运行环境。再比如说很多项目使用的包版本不同，很难同时安装多个版本，此时，可以为不同版本建立不同环境，然后切换到对应版本的环境中工作。

`Anaconda`包括`Python`、`Conda`以及一大堆安装好的科学计算相关工具包，比如：`numpy`、`pandas`等等，开箱即用，非常方便。

`Miniconda`默认只包含`Python`和`Conda`。

换句话说，安装好了`Anaconda`，就相当于同时有了`Python`、`环境管理器`、`包管理器`以及一大堆开箱即用的`科学计算工具包`。

## 安装使用Anaconda

### 安装Anaconda
安装非常简单，注意一下安装目录建议选择磁盘空间较大的地方。官网下载比较慢的话，可以选择清华大学镜像站下载(根据不同操作系统选择相应的包)：  
`https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/?C=M&O=D`

安装成功后，命令行中敲`conda info`，会显示conda的版本和python的版本等详细信息；再敲`conda list`，会列出当前环境下所有安装的包。

### 换成国内镜像源
`conda`和`pip`默认国外镜像源，所以每次安装模块`conda install xxx`或者`pip install xxx`的时候非常慢，换成国内的镜像源会显著加快模块安装速度。

- 修改`Conda`镜像源  
  详细操作见：  
  `https://mirror.tuna.tsinghua.edu.cn/help/anaconda/`

- 修改`pip`镜像源  
  `Linux`系统操作命令如下：
```sh
mkdir ~/.pip
cd ~/.pip
vim pip.conf
```
添加中科大镜像源，内容如下：
```sh
[global]
index-url = https://pypi.mirrors.ustc.edu.cn/simple/
```

不知道你有没有疑问，既然有了`Conda`包管理器，为什么`Anaconda`环境中，可能还需要用`pip`安装包呢？

因为尽管在`Anaconda`下我们可以很方便的使用`conda install xxx`来安装我们需要的依赖，但是`Anaconda`本身只提供部分包，远没有`pip`提供的包多，有时`conda`无法安装我们需要的包，此时可能需要用`pip`将其装到`conda`环境里。

那么`Anaconda`环境下`pip`命令安装的包在哪里呢？会不会影响其他环境呢？

首先要确保用的是本环境的`pip`，这样`pip install xxx`时，包才会安装到本环境中。`Linux`系统可以使用`which pip`来查看当前使用的`pip`是哪个环境的`pip`。

另外需要注意一下：安装特定版本的包，`conda`用`=`，`pip`用`==`。举例来说：
```python
conda install xxx=1.0.0
pip install xxx==1.0.0
```

### 使用Anaconda

安装好了，默认是在`base`虚拟环境下，此时我们从`base`环境复制一份出来，在新环境里工作。

```sh
# 复制base环境, 创建test环境
conda create --name test --clone base

# 激活test环境
conda activate test
```
取消Conda默认激活`base`虚拟环境
```sh
conda config --set auto_activate_base false
```


再列出我本机的所有环境，如下，可见当前有2个环境，当前激活的是`test`环境：
```sh
(test) ➜  ~ conda info -e
# conda environments:
#
base                     /Volumes/300g/opt/anaconda3
test                  *  /Volumes/300g/opt/anaconda3/envs/test
```

`Anaconda`默认安装了`jupyter`，命令行输入：
```sh
jupyter notebook
```
此时会自动弹出浏览器窗口打开`Jupyter Notebook`网页，默认为`http://localhost:8888`。下面会简单介绍一下`Jupyter Notebook`。

这里顺便贴一下`conda`一些常用命令：

- 虚拟环境管理

```bash
# 创建环境，后面的python=3.6是指定python的版本
conda create --name env_name python=3.6

# 创建包含某些包的环境（也可以加上版本信息）
conda create --name env_name python=3.7 numpy scrapy

# 激活某个环境
conda activate env_name

# 关闭某个环境
conda deactivate

# 复制某个环境
conda create --name new_env_name --clone old_env_name

# 删除某个环境
conda remove --name env_name --all

# 生成需要分享环境的yml文件（需要在虚拟环境中执行）
conda env export > environment.yml

# 别人在自己本地使用yml文件创建虚拟环境
conda env create -f environment.yml
```

- 包管理

```bash
# 列出当前环境下所有安装的包
conda list

# 列举一个指定环境下的所有包
conda list -n env_name

# 查询库
conda search scrapys

# 安装库安装时可以指定版本例如：（scrapy=1.5.0）
conda install scrapy

# 为指定环境安装某个包
conda install --name target_env_name package_name

# 更新安装的库
conda update scrapy

# 更新指定环境某个包
conda update -n target_env_name package_name

# 更新所有包
conda update --all

# 删除已经安装的库
conda remove scrapy

# 删除指定环境某个包
conda remove -n target_env_name package_name
```
更多命令请查看官方文档或者查询帮助命令：
```sh
conda --help

conda install --help
```

## 使用Jupyter Notebook

`Jupyter`源于2014年的`ipython`项目，逐渐发展为支持跨所有编程语言的交互式数据科学和科学计算。`Jupyter Notebook`，原名`IPython Notebook`，是`IPython`的加强网页版，一个开源Web应用程序，是一款程序员和科学工作者的编程/文档/笔记/展示软件。上一篇博客[一文上手Python3](https://www.cnblogs.com/bytesfly/p/python.html)就是用`Jupyter Notebook`写的。效果如下：
![](http://img2020.cnblogs.com/blog/1546632/202104/1546632-20210424235722527-129030812.png)

`Jupyter Notebook`可以实时运行代码、渲染`Markdown`，将代码、文本说明和可视化整合在一起，适合数据分析领域的探索性工作，可迭代式地改进代码来改进解决方法。

`Jupyter Notebook`中一对`In Out`会话被视作一个代码单元，称为`cell`。如果`cell`行号前有`*`，表示代码正在运行中。

`Jupyter Notebook`支持两种模式：
- 编辑模式（`Enter`）  
  命令模式下回车`Enter`或鼠标双击`cell`进入编辑模式

- 命令模式（`Esc`)  
  按`Esc`退出编辑，进入命令模式

熟练在这两种模式下工作，再结合一些常用的快捷键，写代码以及文档的效率会大大提高。

### 安装`jupyter_contrib_nbextensions`库

注意：先关闭jupyter后台服务，然后执行下面的命令

```python
pip install jupyter_contrib_nbextensions

jupyter contrib nbextension install --user --skip-running-check
```
重启后，勾选需要的选项如下：  
![](https://img2020.cnblogs.com/blog/1546632/202105/1546632-20210504155939115-948051826.png)

此时就有了代码提示等功能，如下：
![](https://img2020.cnblogs.com/blog/1546632/202105/1546632-20210504160247554-1992201177.png)
