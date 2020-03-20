# personal-note
个人笔记
1. 线程池结构，GC算法，GC垃圾的判定，Young区回收的细节。
2. 注册中心的实现原理
3. 分布式事物的实现方案
4. 秒杀场景的设计
5. Mysql高可用
6. Dubbo通讯过程

-----
快排的思路
反转单链表

-----
项目核心链路要说清楚，上下游也要理解。
生产运维譬如cpu高，load高，内存溢出，堆栈分析过程
tcc中超时问题需要如何设计来避免，类加载机制，锁机制以及原理，redis的应用，对于大key的处理，内存的回收。

https://note.youdao.com/ynoteshare1/index.html?id=40bdf073751319ea82bc535f1fd22bc4&type=note
-----
mysql
索引
索引的底层（页（目录、行格式）、双向链表）
btree和b+tree区别

引擎
innodb
mysaml
锁 https://developer.aliyun.com/article/727076
insert
insert语句会对插入这条记录加行锁，再加行锁之前还会再加一种gap锁，插入意向锁
insert on duplicate 会加ix（表锁）、gap锁、insert intention lock锁
按粒度分
行级锁
表级锁
级别分
共享锁
排他锁
我们正常使用
乐观锁
悲观锁
mvcc
gap锁
next key 间隙锁
insert intention 插入意向锁 数据插入之前会加此锁，轻量的gap锁，多个事务在相同的索引间隙插入时如果不是插入间隙相同的位置就不用互相等待
事务隔离级别
并发造成的几种情况
脏读
修改丢失
不可重复读
幻读
隔离级别
读已提交
读未提交
可重复读
序列化
sql优化
定位问题
show engine innodb status
查看innodb standard monitor输出结果
解决：
1、覆盖索引，减少回表操作
2、SELECT * FROM t2 WHERE id > ? and interest_date=? order by id asc limit 1000
数据量有一天超过了2000万，首次查询的时候特别慢，首次查询id>0。
使用explain发现走的主键这个索引，没有走interest_date。改成强制走interest_date索	引，使用force index，好了(这个方案还可以优化)。
SELECT * FROM t2 force index(idx_date) WHERE id > ? and interest_date=? order by id asc limit 1000
3、死锁
insert on duplicate key update 
事务一 lock_mode x locks gap before rec insert intention waiting，获取gap锁和插入意向锁
事务二有一个sessionId的间隙锁，等待
最后事务一回滚，释放了插入意向锁，导致事务2和事务3同时持有gap锁，等待insert intention，死锁形成
log
binlog：记录数据库表得变更（主从复制）和恢复数据--记载dml sql语句，逻辑变化，赋值和恢复，主从同步数据。是mysql就会有
redolog：加载到内存里面更改之后，一步写入磁盘，同时也会先记录到buffer再写一份redo log，记录着这个页上面得修改--记录物理修改内容（什么页修改了什么），物理变化，做持久化，是innodb引擎产生

-----
JVM
参数：
-xmx 最大堆
-xms 初始堆
-xmn 新生代
-xss 栈
-xx metaspace size 元空间
class文件：
魔数、class文件版本、常量池、访问标志、属性集合、
类加载
加载过程
1、加载连接初始化
2、连接 验证-准备-解析
通过类名获取定义此类的二进制流
将字节流转换成方法区运行时数据结构
内存中生成一个代表该类的class对象，作为访问入口
类加载器
bootstraoClassloader启动类
extensionClassloader扩展类
appClassLoader
双亲委派
自底向上检查类是否被加载
自顶向下尝试加载类
双亲委派模型保证了Java程序的稳定运行，可以避免类的重复加载
jvm内存
虚拟机栈
栈帧
局部变量表
操作数栈：加操作、复制都是入栈和出栈的操作
动态链接：每个栈帧都有一个指向运行时常量池所属方法的引用
方法出口信息：
gcroots：
虚拟机栈中引用的对象
方法区中静态类引用的对象
方法区常量引用的对象
本地方法jni引用的对象
-----
动态代理
new proxyInstance
大概就是把接口复制出来，通过这些接口和类加载器，拿到这个代理类cl。然后通过反射的技术复制拿到代理类的构造函数（这部分代码在Class类中的getConstructor0方法），最后通过这个构造函数new个一对象出来，同时用InvocationHandler绑定这个对象。
-----
线程池如何维护线程对象?
直接通过HashSet持有,Worker对象是在addWorker方法后被新建一个Thread对象运行(Thread对象start之后并没有直接持有,这个Thread被放到了Worker对象作为一个属性持有了),Worker对象的run方法中自己会去自旋取阻塞队列中的Runnable任务.我们只维护了Worker对象,启动Worker对象的线程就在Worker对象中持有.停止时就是遍历了Worker对象取出线程对象,然后调用停止方法等...

阻塞队列的阻塞如何实现的
基础的通知等待模型就可以实现.也可以用ReentrantLock和Condition实现(ArrayBlockingQueue和LinkedBlockingQueue都是基于这个实现的),线程池已不关心实现细节了,只要求是个阻塞队列就可以
进程之间如何通信：
1、管道通信（PIPE）
2、消息队列
3、共享内存
4、信号量
5、socket通信
6、命名管道（FIFO）
线程之间通信：
1、volatile内存共享
2、wait/notify
3、lock（countdownlatch、cyclicbarrier）
-----
显式关闭gap锁
将参数innodb_locks_unsafe_for_binlog设置为1
-----
锁膨胀
无锁状态
没有对资源进行锁定，所有线程都能访问并修改同一个资源（cas）01
偏向锁
一段同步代码一直被一个线程访问，线程会自动获取锁，降低获取锁的代价
markWord或记录线程id，所以线程进入和推出的时候就不会在用cas加锁，直接检测
markword，减少了不必要的cas操作 01
轻量级锁
当偏向锁遇到其他线程竞争的时候，持有偏向锁的线程会释放锁，锁会变成轻量级锁
，其他线程进行自旋，自旋超过一定次数，升级为重量级锁 00
重量级锁
别的线程进行阻塞 10
---
分布式事务
2pc（两阶段提交）
准备阶段：协调者问各个参与事务是否成功，业务返回执行结果处于未提交
提交阶段：都执行成功，协调这告知各个业务，让各个业务提交
（同步阻塞、数据不一致、没有容错、协调者出问题）
tcc（补偿机制，三阶段提交）
try阶段：对业务系统做检测、资源预留
confirm：对业务做确认提交
cancle：业务执行出错，需要回滚，业务执行取消、预留资源释放
（TCC服务在实现时应当允许空回滚的执行）
本地消息表
1、业务插入数据之后、向本地消息表中写入一个消息、确保一定能写入
2、然后将本地消息发送到消息队列中（kafka）
3、另一个业务接收到消息、自己本地插入消息、同时执行业务操作、如果这个消息被
执行过了，就会回滚（不会重复发消息）
4、b执行成功，更新自己表和a系统消息表的状态
5、b失败了，a系统就会定时扫描自己的消息表，再次发送到b
mq事务消息
1、a系统发送一个prepare消息到mq，失败直接不用执行
2、a执行本地事务，成功发送确认消息到mq，失败就告诉mq回滚
3、b接收确认消息，执行本地事务
4、mq会定时轮询所有prepare消息，然后回调a，确认一下是不是本地事务失败了
5、b事务失败，不断重试。如果b回滚，也要通知a回滚
最大努力通知
1、a本地事务执行完、发送mq
2、专门消费最大努力通知服务，这个服务消费mq写入数据库，然后调用系统b
3、b执行成功就ok，失败了 重试，设置几次

---
redis
优点：
数据类型丰富
持久化
集群
多线程io多路复用
支持lua脚本、序列化
单线程，避免了多线程频繁上下分切换
定期删除+惰性删除
内存淘汰机制
volatile-lru：已设置过期最近最少使用
volatile-ttl：将要过期的数据
volatile-random:设置国企时间的随机
allkeys-lru：所有key中选则数据淘汰
allkey-random:
no-evivtion:禁止删数据
4.0之后
volatile-lfu：从设置过期时间挑选最不经常使用
allkeys-lfu
io多路复用：
文件事件处理器，监听多个socket，根据文件事件分派器，选则对应的事件处理器

持久化
快照rdb
aof
缓存雪崩
缓存大面试失效，请求落在数据库
缓存穿透：
请求缓存中不存在数据，落在数据库
布隆过滤器，所有可能存在的数据hash到一个足够大的bitmap中，一个一定不存在的会被这个bitmap拦截
如果查询结果为空，让然进行缓存，过期时间比较短
大key处理：
1、用scan定位到他，然后java代码删除
2、lazyfree机制机制，放在子线程bio中单独处理
redis写操作五个过程：
c端向s端发送写操作（数据存在c端内存中）
s端收到请求数据（数据存在s端内存中）
s端调用write，将数据写入磁盘（数据在内存的缓冲区）
操作系统讲缓冲区的数据一道磁盘控制器上（数据在磁盘缓存中）
磁盘控制器将数据写到磁盘物理介质中（数据在磁盘中）
---
kafka、rabbimq、activeMq、rocketMq
kafka的cluster从集群获取分区leader
producter将消小发送给leader
leader将消息写入本地文件
follow从leader中pull消息
写入本地后发送leader ACK
然后leader接收到之后像producer发送ack
kafka的高可用


--
influxdb push  Prometheus pull
---
分布式锁
zookeeper：每个客户端对某个方法进行加锁时，在zookeeper上与该节点的目录中，生成一唯一的瞬时有序节点，判断节点中序号最小的就是获取锁的
redis：senx
---
红黑树
性质1：节点是红色或黑色。
性质2：根节点是黑色。
性质3：每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
性质4：从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。
---
spring事务传播行为
REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
SUPPORTS：以非事务方式运行，如果当前存在事务，则抛出异常。
MANDATORY以非事务方式运行，如果当前存在事务，则抛出异常。
REQUIRES_NEW：以非事务方式运行，如果当前存在事务，则抛出异常。
NOT_SUPPORTED:以非事务方式运行，如果当前存在事务，则抛出异常。
NEVER:以非事务方式运行，如果当前存在事务，则抛出异常。
---
threadLocal
并不能解决多线程共享变量的问题，用于存放线程上下文变量，方便同一线程多次读取
实际变量存在threadLocalMap中
内存泄漏：
threadlocal get/set 会清除key（key是弱引用，value是强引用）为null的entry，可以用remove
---
自我介绍
2018年毕业于渤海大学，（但是2017年开始工作、经验算是三年，按毕业时间是2年）先后在简理财和人人贷，现在是在职。人人贷是先后在医美事业部（2019.10月），然后转到集团风控事业部，现在风控项目的整个流程大概就是接入下面各个事业部类似于 人人贷、好分期进行风控整合，
简理财就是款理财软件，互联网金融，当时主要负责消息通知和计息方面的东西。
---
java基础
字节码（供虚拟机理解的代码--.class文件）
java是编译与解释并存的（java代码友编译器编译成字节码，再由虚拟机解释执行）


