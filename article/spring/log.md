## Java项目中使用log4j和slf4j实现日志记录

## 什么是log4j？
>Log4j是Apache的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记
录器、UNIX Syslog守护进程等；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。
最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

## 什么是slf4j？
> SLF4J，即简单日志门面（Simple Logging Facade for Java），不是具体的日志解决方案，它只服务于各种各样的日志系统。按照官方的说法，
SLF4J是一个用于日志系统的简单Facade，允许最终用户在部署其应用时使用其所希望的日志系统。

## 常用的日志组合:slf4j+logback

## 项目集成(环境:Spring+SpringMVC+Maven)
- 1、在pom文件中添加依赖
````xml
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    
    <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>${log4j.version}</version>
    </dependency>
````
- 2、添加log4j配置文件(我这里放在了resources下的config文件夹下)
````xml
    #所有日志
    log4j.rootLogger = DEBUG,stdout,D,E,F
    
    #控制台输出
    log4j.appender.stdout = org.apache.log4j.ConsoleAppender    
    log4j.appender.stdout.Target = System.out    
    log4j.appender.stdout.Threshold = INFO  
    log4j.appender.stdout.layout = org.apache.log4j.PatternLayout    
    log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss} %l%m%n    
    
    log4j.appender.D = org.apache.log4j.DailyRollingFileAppender    
    log4j.appender.D.File = ${webApp.root}/xzyc_logs/log.log    
    log4j.appender.D.Append = true    
    #优先级高于等于INFO级别（如：INFO、WARN、ERROR）的日志信息将可以被输出。这样非常消耗服务器资源，正式部署应该调高优先级
    log4j.appender.D.Threshold = INFO     
    log4j.appender.D.layout = org.apache.log4j.PatternLayout    
    log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n    
    
    log4j.appender.E = org.apache.log4j.DailyRollingFileAppender    
    log4j.appender.E.File =${webApp.root}/xzyc_logs/warn.log 
    log4j.appender.E.Append = true
    #设置为warn，输出warn和error级别的信息    
    log4j.appender.E.Threshold = WARN     
    log4j.appender.E.layout = org.apache.log4j.PatternLayout    
    # 2018-01-23 10:37:15 【%t: 输出产生该日志事件的线程名:输出自应用启动到输出该log信息耗费的毫秒数】
    # - %p: 输出日志信息优先级，即DEBUG，INFO，WARN，ERROR，FATAL,这里为 WARN，ERROR，FATAL
    # %m: 输出代码中指定的消息 %n 换行
    #添加类+代码行数[ %c:%L ]
    log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %c:%L ] - [ %p ]  %m%n 
````
- 3、在项目的web.xml文件中添加log4j配置
````xml
    <!--日志配置-->
    <context-param>
        <param-name>log4jConfigLocation</param-name>
        <param-value>classpath:config/log4j.properties</param-value>
    </context-param>
    <!--给日志配置路径-->
    <context-param>
        <param-name>webAppRootKey</param-name>
        <param-value>webApp.root</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
    </listener>
````
- 4、在类中添加日志
```java
    // 导入slf4j类  
    import org.slf4j.Logger;  
    import org.slf4j.LoggerFactory;  
      
    // 添加slf4j日志实例对象  
    private final static Logger logger = LoggerFactory.getLogger(Test.class);  
      
    // 输出日志  
    logger.warn("输出日志");  
```
-5、集成完成，启动项目，可发现在target下的项目目录下发现日志文件。

## 什么时候应该记录日志？
- 1、修改（包括新增）操作必须打印日志，大部分问题都是修改导致的。数据修改必须有据可查。
- 2、条件分支必须打印条件值，重要参数必须打印 尤其是分支条件的参数，打印后就不用分析和猜测走那个分支了，很重要！
- 3、数据量大的时候需要打印数据量，前后打印日志和最后的数据量，主要用于分析性能，能从日志中知道查询了多少数据用了多久。
    这点是建议。自己视情况而决定是否打印，我一般建议打印。

## 参考博客
[在Java项目中如何使用log4j和slf4j实现日志打印](http://blog.csdn.net/xiao_mengxi/article/details/54910450)    
[编码习惯之日志建议](http://blog.didispace.com/cxy-wsm-zml-4/)    
