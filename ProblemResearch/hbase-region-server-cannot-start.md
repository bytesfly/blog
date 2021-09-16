# 使用ClouderaManager管理的HBase的RegionServer无法启动排查

## 问题概述
"新冠期间"远程办公，需要重新搭建一套ClouderaManager(CM)开发环境，一位测试同事发现HBase的RegionServer无法启动，在CM界面上启动总是失败，观察一下日志，也没有什么明显的报错。我就专门看了一下。

## 排查思路
1. 因为有opentsdb在读写Hbase Region Server，我一开始怀疑RegionServer启动过程中在恢复一些数据，这个时候就有组件对它读写操作，可能压力较大起不来。后来停掉了opentsdb，依然如此，日志也没有明显报错，打着打着就断了，再看进程就没了。

2. 后来我在界面上又重启了一下，迅速 <font color='red'>`jps -mlv`</font>命令查看一下启动参数，这一看就明白了居然给的<font color='red'> `堆内存50MB`</font>，难怪起不来，启动过程中应该就<font color='red'>`OOM`</font>了，很快，再执行一次<font color='red'>`jps -mlv`</font>命令 这个<font color='red'>`HRegionServer`</font>进程已经退出了。

3. 于是我在网上搜了一下，果然<font color='red'>`ClouderaManager(CM)`</font>给HBase默认堆内存50M，豁然开朗。

## 解决


![修改HRegionServer堆内存配置](https://img2018.cnblogs.com/blog/1546632/202002/1546632-20200215132131196-898943847.png)

根据实际情况修改一下HMaster、HRegionServer堆内存大小，在界面上重启，我这次用<font color='red'>`jps -mlv`</font>命令观察一下，配置生效了，然后看日志，正常启动中，至此，问题解决。

## 总结

有些时候 程序一启动就挂掉，而且没有什么明显报错日志，可能要观察一下程序的启动参数等。
比如说内存给的太小，程序压根就不能正常启动(OOM异常退出)；
或者内存给的太大，向操作系统申请内存失败直接被<font color='red'>kill</font>掉。


