# Spring与Quartz的整合实现定时任务调度
## Quartz介绍 
Quartz 是一种功能丰富的，开放源码的作业调度库，可以在几乎任何Java应用程序集成 - 从最小的独立的应用程序到规模最大电子商务系统。
Quartz可以用来创建简单或复杂的日程安排执行几十，几百，甚至是十万的作业数 -  作业被定义为标准的Java组件，可以执行几乎任何东西，
可以编程让它们执行。 Quartz调度包括许多企业级功能，如JTA事务和集群支持
如果应用程序需要在给定时间执行任务，或者如果系统有连续维护作业，那么Quartz是理想的解决方案。

## 业务场景
    用积分来区分店铺等级，如果半年内无违规记录，等级自动升级，这时候就可以使用定时任务调度工具，
    在每天凌晨机器资源相对空闲的时候开启定时任务去更新店铺积分

## 整合步骤
- 在pom文件中引入依赖
````xml
    <dependency>
      <groupId>org.quartz-scheduler</groupId>
      <artifactId>quartz</artifactId>
      <version>2.2.3</version>
    </dependency>
````
- 创建定时任务
```java
    @Component
    public class GradeTaskQuartzManager {
        private static Scheduler scheduler = null;
        private static JobDetail jobDetail;
        private static final String JOB_NAME = "GRADE_TASK";
        private static final String TRIGGER_GROUP_ID = "GRADE_TASK_TRIGGERGROUP_ID";
        private static final String TRIGGER_GROUP_NAME = "GRADE_TASK_TRIGGERGROUP_NAME";
    
        /**
         * 调度器
         *
         * @return
         */
        public static Scheduler getScheduler() {
            try {
                if (scheduler != null) {
                    return scheduler;
                }
                // SchedulerFactory sf = new StdSchedulerFactory();
                scheduler = StdSchedulerFactory.getDefaultScheduler();
                return scheduler;
            } catch (SchedulerException e) {
                e.printStackTrace();
            }
            return null;
        }
    
        /**
         * 初始化定时任务
         */
        public void startService() {
            // 创建一个JobDetail实例
            jobDetail = JobBuilder.newJob(GradeTaskJob.class).withIdentity(JOB_NAME, "group1").storeDurably().build();// 不写组默认
            // DEFAULT组
            try {
                // 创建Scheduler实例
                Scheduler scheduler = GradeTaskQuartzManager.getScheduler();
                // 加入一个任务到Quartz框架中, 等待后面再绑定Trigger,
                // 此接口中的JobDetail的durable必须为true
                scheduler.addJob(jobDetail, false);
                // System.out.println(broadcastTask.getId()+"===================================");
                // 创建一个Trigger实例
                CronTrigger trigger = (CronTrigger) TriggerBuilder.newTrigger()
                        .withIdentity(TRI GGER_GROUP_ID, TRIGGER_GROUP_NAME)// 定义标识符
                        .forJob(jobDetail)
                        .withSchedule(CronScheduleBuilder.cronSchedule("0 30 2 * * ? ")).build();//定时策略为每日的凌晨俩点半0 30 2 * * ?
    
                scheduler.scheduleJob(trigger);// 返回Date
                scheduler.start();
            } catch (SchedulerException e) {
                e.printStackTrace();
            }
        }
    }
```

- job
````java
    public class GradeTaskJob implements Job {
        private IShopService shopService;
        private IPlatFormLogService platFormLogService;
        public GradeTaskJob(){
            shopService = SpringContextBean.getBean(IShopService.class);
            platFormLogService = SpringContextBean.getBean(IPlatFormLogService.class);
    
        }
        @Override
        public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
            // 打印当前的执行时间，格式为2017-01-01 00:00:00
            Date date = new Date();
            SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            System.out.println("---------------------------执行定时任务开始------------------------------");
            List<Integer> ids = shopService.getGradeShop();
            for (Integer id:ids) {
                shopService.shopGradeA(id);
                System.out.println("更新店铺Id"+id);
            }
    
            TbPlatformLog platformLog = new TbPlatformLog(TbPlatformLog.LOG_TYPE_CODE_LOG_TASK,"执行定时任务,更新店铺数量:"+ids.size(),-1,new Date());
            platFormLogService.insertPlatFormLog(platformLog);
            System.out.println("执行定时任务啦......."+sf.format(date));
            System.out.println("--------------------------执行定时任务结束-------------------------------");
        }
    }
````

- 创建 SpringContextBean
````java
    public class SpringContextBean {
    
        private static ApplicationContext context;
    
    /*	static{
    		//context=new ClassPathXmlApplicationContext("classpath:spring/applicationContext-*.xml");
    		context=ContextLoader.getCurrentWebApplicationContext();
    	}*/
    
    
        /**
         * 从静态变量ApplicationContext中取得Bean, 自动转型为所赋值对象的类型.
         */
        @SuppressWarnings("unchecked")
        public static <T> T getBean(String name) {
            checkApplicationContext();
            return (T) context.getBean(name);
        }
    
        /**
         * 从静态变量ApplicationContext中取得Bean, 自动转型为所赋值对象的类型.
         */
        @SuppressWarnings("unchecked")
        public static <T> T getBean(Class<T> clazz) {
            checkApplicationContext();
            return (T) context.getBean(clazz);
        }
    
        private static void checkApplicationContext() {
            if (context == null) {
                throw new IllegalStateException("applicaitonContext未注入,请在applicationContext.xml中定义SpringContextHolder");
            }
        }
    
        public static void setContext(ApplicationContext context) {
            SpringContextBean.context = context;
        }
    }

````
- 在Spring容器创建的时候调用定时任务
````java
    @Component
    public class ApplicationBoot implements ApplicationContextAware, ServletConfigAware, InitializingBean,
            ApplicationListener<ApplicationEvent> {
    
        private static ApplicationContext applicationContext;     //Spring应用上下文环境
    
        @Autowired
        private GradeTaskQuartzManager gradeTaskQuartzManager;
    
        // 第一执行此函数，因为初始化操作越早越好，建议在此方法中进行初始化操作
        //这里进行定时任务启动
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            SpringContextBean.setContext(applicationContext);
            //执行定时任务
            gradeTaskQuartzManager.startService();
        }
    
        // 再次执行此函数
        @Override
        public void afterPropertiesSet() throws Exception {
    
        }
    
        // 第二执行此函数
        @Override
        public void setServletConfig(ServletConfig servletConfig) {
    
        }
    
        // 最后执行此函数，但此函数会执行多次
        // 经进测试，onApplicationEvent这个方法会执行多次，所以不建议在此方法中执行初始化操作
        @Override
        public void onApplicationEvent(ApplicationEvent applicationEvent) {
    
        }
    }
````

- 集成完成
