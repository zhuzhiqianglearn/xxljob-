

# xxl-job 使用，集群和高可用

这里使用springboot代码演示

#### xxl-job简介，参考github链接

> https://github.com/xuxueli/xxl-job/

#### 一、简单上手

1.下载github 的代码

结构如下

![image-20210204182927660](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204182927660.png)



2.打开 xxl-job/doc/db/tables_xxl_job.sql 数据库文件，并在本地库运行

![image-20210204183132379](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204183132379.png)



3.打开xxl-job-admin工程的配置文件，修改服务端口和数据库连接信息（邮箱其他的可以省略，需要使用在进行配置）

![image-20210204183254609](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204183254609.png)



4.然后启动admin 工程，访问：http://localhost:8080/xxl-job-admin/，出现下面的页面，代表成功

![image-20210204183457541](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204183457541.png)



5.修改xxl-job-executor-sample-springboot工程的配置文件

![image-20210204183741356](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204183741356.png)

```
xxl.job.admin.addresses：注册的地址
xxl.job.executor.port=9998 tcp一个用来执行任务的端口
```

6.我用的是新版的xxl-job（老版的得继承JobHander的一个类，然后在写上注解的名称）

只需在方法名称上添加 @XxlJob 注解即可，value就是我们job的名称

```
@Component
public class DemoHander {
    private static Logger logger = LoggerFactory.getLogger(SampleXxlJob.class);
    private String port;

    @XxlJob(value = "mytest")
    public ReturnT<String> mytest(String param) throws Exception {
        XxlJobLogger.log("XXL-JOB, Hello World.");
        System.out.println(port+"-----------");
        XxlJobLogger.log(param+"beat at:");
        return ReturnT.SUCCESS;
    }

}
```

7.然后启动我们的xxl-job-executor-sample-springboot工程

8.新增一个执行器（我使用的是手动的，因为便于扩展）

<!--步骤4，我选的是手动，因为便于增加节点，选择自动也是可以的-->

<!--步骤5，新版本一定要加上http:// 不然会报错-->

![image-20210204184638290](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204184638290.png)



9.新增一个任务



![image-20210204185003640](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204185003640.png)

参数说明：

    - 执行器：任务的绑定的执行器，任务触发调度时将会自动发现注册成功的执行器, 实现任务自动发现功能; 另一方面也可以方便的进行任务分组。每个任务必须绑定一个执行器, 可在 "执行器管理" 进行设置;
    - 任务描述：任务的描述信息，便于任务管理；
    - 路由策略：当执行器集群部署时，提供丰富的路由策略，包括；
        FIRST（第一个）：固定选择第一个机器；
        LAST（最后一个）：固定选择最后一个机器；
        ROUND（轮询）：；
        RANDOM（随机）：随机选择在线的机器；
        CONSISTENT_HASH（一致性HASH）：每个任务按照Hash算法固定选择某一台机器，且所有任务均匀散列在不同机器上。
        LEAST_FREQUENTLY_USED（最不经常使用）：使用频率最低的机器优先被选举；
        LEAST_RECENTLY_USED（最近最久未使用）：最久未使用的机器优先被选举；
        FAILOVER（故障转移）：按照顺序依次进行心跳检测，第一个心跳检测成功的机器选定为目标执行器并发起调度；
        BUSYOVER（忙碌转移）：按照顺序依次进行空闲检测，第一个空闲检测成功的机器选定为目标执行器并发起调度；
        SHARDING_BROADCAST(分片广播)：广播触发对应集群中所有机器执行一次任务，同时系统自动传递分片参数；可根据分片参数开发分片任务；
        
    - Cron：触发任务执行的Cron表达式；
    - 运行模式：
        BEAN模式：任务以JobHandler方式维护在执行器端；需要结合 "JobHandler" 属性匹配执行器中任务；
        GLUE模式(Java)：任务以源码方式维护在调度中心；该模式的任务实际上是一段继承自IJobHandler的Java类代码并 "groovy" 源码方式维护，它在执行器项目中运行，可使用@Resource/@Autowire注入执行器里中的其他服务；
        GLUE模式(Shell)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "shell" 脚本；
        GLUE模式(Python)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "python" 脚本；
        GLUE模式(PHP)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "php" 脚本；
        GLUE模式(NodeJS)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "nodejs" 脚本；
        GLUE模式(PowerShell)：任务以源码方式维护在调度中心；该模式的任务实际上是一段 "PowerShell" 脚本；
    - JobHandler：运行模式为 "BEAN模式" 时生效，对应执行器中新开发的JobHandler类“@JobHandler”注解自定义的value值；
    - 阻塞处理策略：调度过于密集执行器来不及处理时的处理策略；
        单机串行（默认）：调度请求进入单机执行器后，调度请求进入FIFO队列并以串行方式运行；
        丢弃后续调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，本次请求将会被丢弃并标记为失败；
        覆盖之前调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，将会终止运行中的调度任务并清空队列，然后运行本地调度任务；
    - 子任务：每个任务都拥有一个唯一的任务ID(任务ID可以从任务列表获取)，当本任务执行结束并且执行成功时，将会触发子任务ID所对应的任务的一次主动调度。
    - 任务超时时间：支持自定义任务超时时间，任务运行超时将会主动中断任务；
    - 失败重试次数；支持自定义任务失败重试次数，当任务失败时将会按照预设的失败重试次数主动进行重试；
    - 报警邮件：任务调度失败时邮件通知的邮箱地址，支持配置多邮箱地址，配置多个邮箱地址时用逗号分隔；
    - 负责人：任务的负责人；
    - 执行参数：任务执行所需的参数；



10.点击启动执行（查看执行日志）

![image-20210204185621882](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204185621882.png)

![image-20210204185837527](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204185837527.png)



## 二、任务集群搭建



1.重复  修改xxl-job-executor-sample-springboot工程的配置文件 这个步骤 修改服务端口和xxl.job.executor.port

2.然后启动服务

3.打开任务电镀中心页面，编辑刚刚创建的执行器，在机器地址这里，将新的节点添加进去就可以

![image-20210204190243708](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204190243708.png)

4.修改刚刚创建的任务，策略改为轮询，然后查看自己启动的服务即可

![image-20210204190629273](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204190629273.png)

![image-20210204190751006](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204190751006.png)

![image-20210204190808103](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204190808103.png)



## 三、调用服务的高可用



1.类似于任务的集群方式，只需将xxl-job-admin 的配置文件 修改下服务端口即可（数据库一定保持一致）

2.修改本机的host

![image-20210204191153197](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204191153197.png)

3.配置nginx，然后启动

```
 upstream  backServer{
            server 127.0.0.1:8080 weight=1;
            server 127.0.0.1:8079 weight=1;
    }
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            add_header backendIP $upstream_addr; //显示nginx跳转的地址
            proxy_pass http://backServer;
            index  index.html index.htm;
        }
    。。。
```

![image-20210204191353695](/Users/zhuzhiqiang/Library/Application Support/typora-user-images/image-20210204191353695.png)

4.修改xxl-job-executor-sample-springboot 配置文件（注册地址修改为如下面的路径）

```
xxl.job.admin.addresses=http://www.myxxljob.com/xxl-job-admin
```

5.重启所有的项目即可





















工单表

es索引，类信息  nacos/mysql

json数组   mysql 表

节点表

公告

排班