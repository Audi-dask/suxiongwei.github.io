## 将tomcat添加为linux系统服务

- 1、复制catalina.sh作为系统服务脚本
cp /usr/java/tomcat/bin/catalina.sh /etc/init.d/tomcat    #重命名的tomcat为以后的服务名

- 2、修改脚本
vi /etc/init.d/tomcat</br>

在脚本较前面的位置加下面两行注释</br>
````jshelllanguage
    #chkconfig:2345 10 90
    #description:Tomcat service
````
> 第一行是服务的配置：第一个数字是服务的运行级，2345表明这个服务的运行级是2、3、4和5级（Linux的运行级为0到6）；第二个数字是启动优先级，数值从0到99；第三个数
  是停止优先级，数值也是从0到99。 
  第二行是对服务的描述
  如果该注释有误，在添加服务时会出现“tomcat不支持chkconfig”的错误提示

- 3、在脚本中设置环境变量(修改为自己环境下的对应路径)
````jshelllanguage
    CATALINA_HOME=/usr/java/tomcat
    JAVA_HOME=/usr/java/jdk1.7.0
````

- 4、添加脚本的可执行权限
````jshelllanguage
    chmod 755 /etc/init.d/tomcat
````

- 5、添加为系统服务
````jshelllanguage
    chkconfig --add tomcat
````

- 6、查看系统服务列表
````jshelllanguage
    chkconfig --list
````

- 7、设置为开机自动启动
````jshelllanguage
    vi /etc/rc.local
````
添加startup.sh的路径 /usr/java/tomcat/bin/startup.sh

- 8、配置完成，进行测试
````jshelllanguage
    启动tomcat： service tomcat start
    停止tomcat： service tomcat stop
````
    
## 运行截图
![](https://github.com/suxiongwei/suxiongwei.github.io/tree/master/images/tomcat_service.png)    

## 参考博客
[将tomcat添加为linux系统服务](http://blog.csdn.net/zfl589778/article/details/51333442)
