SSH框架整合

SSH三大框架整合
---------------

鉴于我的MyEclipse版本是最新的,你的版本可能比较低,有问题可以问我 \#\#
第一步建立数据库 ,这个是比较重要的,之后的hibernate逆向工程要用到
在mysql中建立一个student数据库,并创建表login_info,student_info
创建student表(当然可以创建你自己的表,自己的字段,你喜欢就好啦)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
create table student.student_info
(
  xhao   varchar(11) null,
  xming  varchar(11) null,
  xbie   varchar(11) null,
  nling  int         null,
  xyuan  varchar(11) null,
  bji    varchar(11) null,
  zhye   varchar(11) null,
  dhua   varchar(11) null,
  yxiang varchar(11) null
);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

第二步创建工程
--------------

打开MyEclipse创建一个web project,当然你的可能还有一个dynamic web
project,这些项目可以相互转化,你喜欢就好

![](media/46a6e194bed5f36e9ab41f89b8144fdf.png)

![](media/17d7d43c144a6a92404ef4054f7a2fd9.png)

>   下面是编译完成的二进制文件位置

![](media/1deb57f6b38a4b1b5364bca313e49636.png)

![](media/75d76295f4ec4b146cef87ef158fce92.png)

![](media/1fb65e04887f2ed57e722f8cec632ad5.png)

\> 默认就可以,点击finish,项目创建完成 完成后的目录

![](media/87b62b4e85fbae38013d23d00ce37d60.png)

\#\# 第三步添加ssh三大框架的依赖 \#\#\# 添加spring的依赖
选中你的工程点击菜单栏上的project,如图所示,或者右键单击菜单的configure facets
(这里低版本的可能在菜单栏的MyEclipse选项下找facets有关的选项)

![](media/c7a1e303015bcb13a3f0d3bcc82131c9.png)

下面配置的过程就按图示来即可 下图选择spring的版本,老师用的3.0,也是你喜欢就好

![](media/797c82cfde875dabc499291b8aa74e2b.png)

![](media/106ebf59a5200a9df6479df930a81508.png)

![](media/50c8915caa9ca484d9ac34c0b0759e2d.png)

配置hibernate
-------------

选中你的工程点击菜单栏上的project,像配置sping一样,或者右键单击菜单的configure
facets
然后可以在src里创建student实体类的包entity或者model(为将来的逆向工程准备)也可以同时顺便创建action,service,service,impl,dao包

![](media/fb187873a40d4862c7d1da2cc813a80a.png)

![](media/9fb3efa8e8cda10a5ad17bf377bf1d06.png)

![](media/847e81f11eab55db0d0d21def30ed1ed.png)

![](media/fc2934942b0e9a32bbd4856a7c87b6f1.png)

![](media/7ee8b3ead7855c9ee3eb4fb5ad194fbc.png)

逆向工程
--------

点击切换到databaseexplorer视图

![](media/8eb5058b2698f435f1018a5d895c7e16.png)

\> 在这里新建一个数据库连接 点击New

![](media/3015a1ec681d1fd4e4d092add4360b13.png)

这一这里的jar包驾驶你的mysql-connetor的jar包

![](media/4786ba7f87c275e867b2516a04b81b88.png)

连接成功

![](media/e5f75d06d3d1ef082178f8cb6c7384c3.png)

\#\# 逆向工程 双击打开你的mysql连接,进行逆向工程

![](media/8f5ccac33dc3bea76fe0ab9ed4b5a735.png)

>   在你的表上右键,找到逆向工程菜单并点击

![](media/eb8f5b75197d09d1669c9e6d8d11c014.png)

按图中选中的就好

![](media/11da32f667dc3335aadf1c0e6e64d8b0.png)

然后一路下一步,下一步

![](media/36555e4c497581947a8759436b029e3b.png)

![](media/489571d76c370092500898f0656b4c4b.png)

然后切换到MyEclipse视图 看到entity包里面已经有了

![](media/17cc25c87ee0fcbfbc68c66a0fc9f4b7.png)

![](media/a1da242c8b493886a7d0e7fb755f1940.png)

\#\# 配置struts
选中你的工程点击菜单栏上的project,像配置sping一样,或者右键单击菜单的configure
facets install struts2

![](media/77c807bc2d7341246f2c0279dcb60ab4.png)

![](media/a1b46a9e426ef911cb72c8431558e4a8.png)

>   如下图选中 core Facets spring web 后面的dwr可选可不选(ajax框架)

![](media/0f90afbfdae5f65989cc664a3d74c19f.png)

>   配置完了struts可以写action包里的类,也可以写service包里的类,看你的喜好了,我选择写service里的类

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
package service;

import java.util.List;
import entity.StudentInfo;

public interface QryService {

    public List<StudentInfo> getAll();

    public boolean del(Integer num);

    public StudentInfo findXshxx(Integer num);

    public boolean update(StudentInfo xsh);

    public void saveStd(StudentInfo xsh);

}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

然后编写service的实现类

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
public class QryServiceImpl implements QryService {

    private StudentInfoDAO stdinfoDao;
    private StudentInfo stdinfo, stdinfo1;
    private List<StudentInfo> lst;
......

}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

之后可以将老师的jsp放在webroot文件夹里面是前端页面

![](media/7f4ea44baffbe6fa3ad7a46363c4dc32.png)

最后配置action
在action包中配置相应的StudentAction类并将相应的action写入struts.xml中
ssh的配置文件application.xml web.xml都是自动生成的,几乎不用配置 下面是文件结构

![](media/59472df1209f79ee2bb25a46c7ad9d49.png)

大致就像老师的ssh架构示意图 这里配好了struts,写好了前后端代码,已经配置成功了
之首就是打开tomcat服务器,不停的调试运行,最终运行失败,不过也不担心,因为老师的也不一定对

其实上面讲的就是实验二的作业,你可以好好看一看,有问题问我,我随叫随到\~
