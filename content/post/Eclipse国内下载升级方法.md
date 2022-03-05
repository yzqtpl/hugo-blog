# Eclipse国内下载升级方法



我们在国内从官网下载Eclipse以及插件非常慢，那么，有没有方法变快呢？

有，那就是使用国内的公开镜像源替换官方源。

## 1 下载Eclipse

首先，我们看一个链接地址：

<http://mirror.bit.edu.cn/eclipse/technology/epp/downloads/release/juno/SR1/eclipse-jee-juno-SR1-win32.zip>

其中， [http://mirror.bit.edu.cn](http://mirror.bit.edu.cn/) 代表一个国内的镜像源，后面的路径基本是固定的，所以，如果需要下载一个新的版本时，可以到：

<http://mirror.bit.edu.cn/eclipse/technology/epp/downloads/release/>

路径下去找时间戳最新的那个文件夹，然后，去下载最新的版本。

常用的几个速度很快的源：

中国科学技术大学(5.6MB/s) <http://mirrors.ustc.edu.cn/eclipse/>

北京理工大学（600KB/s） <http://mirror.bit.edu.cn/eclipse/>

大连东软信息学院(400KB/s) <http://mirrors.neusoft.edu.cn/eclipse/>

## 2 安装插件

同样，需要把里面的源改为速度最快的源：

> help -> Install New Software… -> Available Software Sites

![screenshot_2017-01-19_23-30-11.png](https://images2015.cnblogs.com/blog/717724/201701/717724-20170119235841140-1784404819.jpg)

把 <http://download.eclipse.org/> 替换为 <http://mirrors.ustc.edu.cn/eclipse> 。

![screenshot_2017-01-19_23-34-51.png](https://images2015.cnblogs.com/blog/717724/201701/717724-20170119235841468-1234070040.jpg)