## 运维技能

### 1. CDN

你有个web服务，部署在了北京的机房，那么北京的用户访问速度肯定是大于其他区域的用户的。那么对于一些高频率访问且要求实时性比较高的内容，如何提高其他地域的用户的访问速度呢？很容易想到的一个办法就是在每个地域都部署一个服务然后服务之间以某种机制进行同步。这就是CDN，全称是内容分布式网络。

对于一般的中小型公司来说，都是租用大的cdn供应商的服务的。除非你特别有钱，能够在全国各个地方都部署自己的服务器，然后构建自己的CDN。

一般情况来说，对于静态内容或者不是很频繁变化的内容都可以通过cdn提供服务，能够在很大情况下提高系统应对并发的能力。

### 2. 负载均衡

负载均衡在服务端领域中是一个很关键的技术。可以分为以下两种。

* 硬件负载均衡
* 软件负载均衡

其中，硬件负载均衡的性能无疑是最优的，其中以F5为代表。但是，与高性能并存的是其成本的昂贵。所以对于很多初创公司来说，一般是选用软件负载均衡的方案。

软件负载均衡中又可以分为四层负载均衡和七层负载均衡。四层负载均衡指的是tcp层，代表的软件为lvs，此软件的创始人为章文嵩博士（就职于阿里），是全世界使用最多的软件负载均衡软件。七层负载均衡的代表则是nginx，是反向代理服务器的一种典型应用。此外，HaProxy兼具四层和七层负载均衡的功能，也是一种很强大的负载均衡软件。

对于LVS，其分为三种模式：

* NAT: 修改数据包destination ip，in和out都要经过lvs。
* DR：修改数据包mac地址，lvs和realserver需要在一个vlan。
* IP TUUNEL：修改数据包destination ip和源ip，realserver需要支持ip tunnel协议。lvs和realserver不需要在一个vlan。

三种模式各有优缺点，目前还有阿里开源的一个FULL NAT是在NAT原来的DNAT上加入了SNAT的功能。

对于Nginx,其本来的功能是一个Web服务器，类似的还有apche httpserver，他们都具有反向代理的功能，基于此功能，便可以提供负载均衡的功能。ngixn的特殊设计，使其可以抗住很高的并发（24G内存可以抗住200万并发）。对于其的优化配置可见https:\/\/github.com\/superhj1987\/awesome-config\/tree\/master\/nginx

阿里开源的tengine是其在nginx开源版本上的做的二次开发，基本涵盖了nginx的收费版本的一些高级功能。比如加入的upstream主动健康检查弥补了nginx在负载均衡方面的不足，使得nginx在lb上变得更加强大。想要深入了解的可以访问[http:\/\/tengine.taobao.org\/](http://tengine.taobao.org/)。

### 3. 应用服务器

目前比较常用的web服务器包括

* Apache http server

  此服务器一般用于在lamp架构中，主要是运行php程序的。当然，现在的php服务有向lnmp转化的趋势。

* Tomcat

  此服务器是典型的java应用服务器，基本上是java后端应用最常用的应用服务器。对于其的优化配置可见https:\/\/github.com\/superhj1987\/awesome-config\/tree\/master\/tomcat。

* IIS

  此服务器是c\#、asp等win系列服务的应用服务器。性能实在不敢恭维。一般来说在传统it行业占有率比较高。


### 4. 数据库

#### mysql

mysql是目前最常用的关系型数据库，支持复杂的查询。但是其负载能力一般，很多时候一个系统的瓶颈就发生在mysql这一点，当然有时候也和sql语句的效率有关。比如，牵扯到联表的查询一般说来效率是不会太高的。

影响数据库性能的因素一般有以下几点：

* 硬件配置：这个无需多说
* 数据库设置：max\_connection的一些配置会影响数据库的连接数, 典型的配置可见：https:\/\/github.com\/superhj1987\/awesome-config\/tree\/master\/mysql。
* 数据表的设计：使用冗余字段避免联表查询；使用索引提高查询效率
* 查询语句是否合理：这个牵扯到的是个人的编码素质。比如，查询符合某个条件的记录，我见过有人把记录全部查出来，再去逐条对比
* 引擎的选择：myisam和innodb两者的适用场景不同，不存在绝对的优劣

抛开以上因素，当数据量单表突破千万甚至百万时（和具体的数据有关），需要对mysql数据库进行优化，一种常见的方案就是分表：

* 垂直分表：在列维度的拆分
* 水平分表：行维度的拆分

此外，对于数据库，可以使用读写分离的方式提高性能，尤其是对那种读频率远大于写频率的业务场景。这里采用master\/slave的方式实现读写分离，前面用程序控制或者加一个proxy层。可以选择使用MySQL Proxy，编写lua脚本来实现基于proxy的mysql读写分离。

现在很多大的公司对这些分表、主从分离、分布式都基于mysql做了自己的二次开发，形成了自己公司的一套分布式数据库系统。比如阿里的Cobar、网易的DDB等。

#### redis

当然，对于系统中并发很高并且访问很频繁的数据，关系型数据库还是不能妥妥应对。这时候就需要缓存数据库出马以隔离对mysql的访问,防止mysql崩溃。
其中，redis是目前用的比较多的缓存数据库（当然，也有直接把redis当做数据库使用的）。redis是单线程基于内存的数据库，读写性能远远超过mysql。一般情况下，对redis做读写分离主从同步就可以应对大部分场景的应用。但是这样的方案缺少ha，尤其对于分布式应用，是不可接受的。目前，redis集群的实现方案有以下几个：

* [redis cluster](http://redis.io/topics/cluster-tutorial):这是一种去中心化的方案，是redis的官方实现。是一种非常“重”的方案，已经不是Redis单实例的“简单、可依赖”了。目前应用案例还很少，貌似国内的芒果台用了，结局不知道如何。
* [twemproxy](https://github.com/twitter/twemproxy)：这是twitter开源的redis和memcached的proxy方案。比较成熟，目前的应用案例比较多，但也有一些缺陷，尤其在运维方面。比如无法平滑的扩容\/缩容，运维不友好等。
* [codis](https://github.com/wandoulabs/codis): 这个是豌豆荚开源的redis proxy方案，能够兼容twemproxy，并且对其做了很多改进。由豌豆荚于2014年11月开源，基于Go和C开发。现已广泛用于豌豆荚的各种Redis业务场景。现在比Twemproxy快近100%。目前据我所知除了豌豆荚之外，hulu、网易等公司也在使用这套方案。当然，其升级项目[reborndb](https://github.com/reborndb/reborn)号称比codis还要厉害。

对于redis的典型配置可见：https:\/\/github.com\/superhj1987\/awesome-config\/tree\/master\/redis。
### 5. Linux性能优化与诊断

由于现在大多数互联网应用都是部署在Linux上的，因此对于Linux系统的优化以及故障诊断是一个很关键的技能。而掌握Linux的shell编写则是掌握Linux性能优化与诊断的前提条件。对于Shell来说，有以下几点需要着重学习：

* 使用vim编写文件

* 使用less、more、tail等查看文件

* 使用Linux的crontab来实现定时调度程序

* Linux下的服务守护进程如何实现?这个可以参考\/etc\/init.d下很多默认自带的service是如何编写的。

* awk和sed是需要掌握的命令，能够快速分析日志文件，定位系统问题。

* 想要后台启动任务，可以通过bg命令或者命令后附带&，当然，使用nohup命令可以产生输出日知道nohup文件中
* 使用tmux工具可以实现多任务会话后台运行

除了上面所述，以下就Linux系统几个关键的性能指标以及诊断、调优方法和命令分别做介绍，也是需要着重掌握的。

* CPU利用率：这是系统非常重要的一个性能指标，表示系统运算的繁忙程度。对于普通的web程序，如果cpu达到200%以上，就说明有耗费cpu的计算或者进程在进行中，需要排除是正常的业务还是其他异常、垃圾回收造成的。一般来说通过下面的命令是可以获取到相关的信息的,命令具体的使用可参见man手册，后续的命令不做具体介绍的同样。

  * uptime: 此命令返回的是系统的平均负荷，包括1min、5min、15min内可以运行的任务平均数量，包括正在运行的任务以及虽然可以运行但正在等待某个处理器空闲的任务。当然，这个值是和cpu核数有关的，双核的机器，load只要小于2也是正常的状况。cpu的情况可以通过查看\/proc\/cpu来获得。

     18:45:46 up 93 days, 14:51, 7 users, load average: 0.00, 0.01, 0.05

  * vmstat：vmstat是一个实时性能检测工具

    * procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu------

       r b swpd free buff cache si so bi bo in cs us sy id wa st

       0 0 0 7887984 1320604 6288252 0 0 0 2 0 1 0 0 100 0 0


            一般来说，id + us + sy = 100,一般认为id是空闲CPU使用率，us 是用户CPU使用率，sy是系统CPU使用率。

    在计算cpu利用率的时候，建议多获取几次，尤其是在脚本里获取时，一般只获取一次是不准确的，建议在脚本里取两次以上并排除掉第一次的数据。 
    * top：




* 内存利用率: 内存利用率也是系统非常重要的一个指标。当发现内存使用率很高的时候，需要判断是必要的使用还是发生了内存泄露，对于前者，则只能通过扩充内存。后者，则需要定位原因并排除。这里需要注意的是：使用free命令时，第一行的信息是针对整个os来说的，因此Buffer和Cache都被计算在了used里面，其实这两部分内存是可以被很快拿来供应用程序使用的。因此，真正反映内存使用状况的是第二行。

  * \/proc\/meminfo

  * \/proc\/slabinfo

  * free



* I\/O利用率：这个指标则是值得是IO设备的使用率。表示系统IO设备的防盲程度。可以通过以下命令观察到。

  * iostat

  * sar



* 网络利用率：网络利用率则表现出系统的网络状况。能看到系统每个连接的状态。这里需要注意的是两种连接状态：TIME\_WAIT和CLOSE\_WAIT。如果系统中这两种连接过多，会直接拖垮整个系统的性能。前者是主动关闭方接收不到ack，一般需要调优内核参数来优化；后者则是被动关闭方接收不到ack，一般是程序没有释放连接造成的。

  * netstat

  * ifconfig



* 系统跟踪：当需要对系统性能做追踪的时候，可以使用top命令查看系统总体的性能指标以及各个进程的状况。也可以使用strace去跟踪每个应用的系统调用情况。

  * top

  * strace



* 调度器调优：系统的调度器对系统的性能也有其重要。

  * 单处理器

  * 对称多处理器\(SMP\)

  * 非一致性内存访问\(NUMA\)

    Linux2.6内核调度器\(多队列调度器\) CHILD\_PENALTY      CREDIT\_LIMIT



* 文件系统调优：选择合适的文件系统，也关系着系统的存储、IO性能。

  * Ext2

  * Ext3（日志 tuner2fs）

  * ReiserFS（Reiserfstune）

  * JFS



* 网络调优-TCP\/IP协议内核参数: 对于Linux系统的性能，其内核的很多参数都关系着系统的性能指标。

  * tcp\_rmem tcp\_wmen

  * tcp\_keepalive\_time

  * ip\_local\_port\_range

  * tcp\_max\_syn\_backlog

  * tcp\_max\_tw\_buckets  netdev\_max\_backlog



这里不得不说的是vmstat命令，vmstat命令是最常见的Linux\/Unix监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。这个命令有两个最大的优势，一个是Linux\/Unix都支持，二是相比top，我可以看到整个机器的CPU,内存,IO的使用情况，而不是单单看到各个进程的CPU使用率和内存使用率\(使用场景不一样\)。

其实一般来说通过使用以下十条命令基本就可以在1分钟内对系统资源使用情况有个大致的了解。

* uptime：

* dmesg \| tail

* vmstat 1

* mpstat -P ALL 1

* pidstat 1

* iostat -xz 1

* free -m

* sar -n DEV 1

* sar -n TCP,ETCP 1

* top


此外，这里有一个本人参与的开源项目，里面收集了很多linux运维需要的shell脚本等。地址见：[https:\/\/github.com\/superhj1987\/useful-scripts](https://github.com/superhj1987/useful-scripts)

### SRE

SRE指的是Site Reliable Engineering\(网站可用性工程\)。这个词起源于Google，是运维更进一层的一个角色，一般是由软件研发工程师转型而来的。SR进一步可以分为：System Reliability、 Service Reliability、 Speed Reliability 以及 Security Reliability。可见，SRE这一职位需要掌握的技能点之广。

在Google的SRE工程师一般需要掌握:算法，数据结构，编程能力，网络编程，分布式系统，可扩展架构，故障排除。比起国内的OPS要求要高得多，一般由软件研发工程师转型而来。是一个比较综合并且高级的职位。

对于这个职位来说，需要扎实掌握计算机科学的相关理论性以及工程性知识，除此之外，充足的工程经验也是必不可少的。

这里的sar,pidstat以及iostat都是sysstat软件套件的一部分，需要单独安装