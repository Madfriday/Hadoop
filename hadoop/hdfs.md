#Welcome to use Hadoop

1.1 大数据者，海量数据之存储者也:  
  存储框架:  
  * HDFS -- 分布式文件存储系统  
  * HBASE -- 分布式数据库系统  
  * KAFKA -- 分布式消息缓存系统  
  * YARN -- 分布式资源调度平台  
  

  运算框架：  
  * MAPREDUCE -- 离线批处理  
  * SPARK -- 离线批处理|实时流式计算  
  * STORM --  实时流式计算  

辅助工具：  
  * FLUME -- 数据采集  
  * SQOOP -- 数据迁移  
  * HIVE -- 数据仓库，将SQL语句翻译为MapReduce运行计算
  * ELASTICSEARCH -- 分布式搜索引擎
  

1.2 什么是Hadoop:
hadoop由3个核心组件组成：  
* 分布式文件系统 -- HDFS  
* 分布式运算框架 -- MapReduce  
* 分布式资源调度平台 -- YARN  

1.3 hdfs运行机制

![hdfs](images/11.png "hdfs")



* hdfs有着文件系统类似的结构：  
1.  有目录结构，顶层目录为/  
2. 系统存放的就是文件  
3. 系统可以提供对文件的创建、删除、修改、查看、移动等功能

* hdfs与普通操作系统的区别：
1. 单机文件系统中存放的文件，是在一台机器的操作系统中  
2. dfs的文件系统会横跨N多的机器  
3. 单机文件系统中存放的文件，是在一台机器的磁盘上  
4. hdfs文件系统中存放的文件，是落在n多机器的本地单机文件系统中（hdfs是一个基于linux本地文件系统之上的文件系统）

* hdfs的工作机制：
1. 客户把一个文件存入hdfs系统中，活动方式会把这个文件切分成多个块后，分布存储在N个服务器上(datanode)  
2. 当多个block块存在多台datanode上时，需要一台机器记录存放规则信息(namenode)  
3. 为了防止分散的块丢失，需要将每一个block指定n个副本存放在datanode上  

1.4 hdfs安装搭建 
  * 准备n台服务器  
  * 一台作namenode，其余作datanode
  * 修改各台主机名以及ip地址 vi /etc/sysconfig/network：  
  
  |主机名|ip地址|
  |---|---|
  |node1|192.168.9.11|
  |slave1|192.168.9.12|
  |slave2|192.168.9.13|
  |slave3|192.168.9.14|
  
  * 从windows中用CRT软件进行远程连接:  
  在windows中将各台linux机器的主机名配置到的windows的本地域名映射文件中：c:/windows/system32/drivers/etc/hosts
 192.168.9.11 node1   
 192.168.9.12 slave1  
 192.168.9.13 slave2  
 192.168.9.14 slave3
  
  * 配置linux服务器的基础环境:  
 关闭防火墙：service iptables stop    
关闭防火墙自启： chkconfig iptables off

* 安装JDK：  
配置环境变量：  
vi /etc/profile，在文件的最后，加入：
export JAVA_HOME=/root/apps/jdk1.8.0_60  
export PATH=$PATH:$JAVA_HOME/bin  
修改完成后，记得 source /etc/profile使配置生效  

* 集群主机域名映射：  
在namenode上，vi /etc/hosts:  
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
 192.168.9.11 node1   
 192.168.9.12 slave1  
 192.168.9.13 slave2  
 192.168.9.14 slave3  
 然后，将hosts文件拷贝到集群中的所有其他机器上  
scp /etc/hosts hdp-02:/etc/  
scp /etc/hosts hdp-03:/etc/  
scp /etc/hosts hdp-04:/etc/  

* 安装hdfs集群：  
上传hadoop到namenode上，修改配置文件：  
核心配置参数：  
1)	指定hadoop的默认文件系统为：hdfs  
2)	指定hdfs的namenode节点为哪台机器  
3)	指定namenode软件存储元数据的本地目录  
4)	指定datanode软件存放文件块的本地目录  

(1) vi /root/apps/hadoop/etc/hadoop/hadoop-env.sh  
export JAVA_HOME=/root/apps/jdk1.8.0_60

(2) vi /root/apps/hadoop/etc/hadoop/core-site.xml 

```
<configuration>  
<property>
<name>fs.defaultFS</name>
<value>hdfs://node1:9000</value>
</property>
</configuration>
```

 
 
(3) vi /root/apps/hadoop/etc/hadoop/hdfs-site.xml
```
<configuration>
<property>
<name>dfs.namenode.name.dir</name>
<value>/root/hdpdata/name/</value>
</property>

<property>
<name>dfs.datanode.data.dir</name>
<value>/root/hdpdata/data</value>
</property>

<property>
<name>dfs.namenode.secondary.http-address</name>
<value>slave1:50090</value>
</property>

</configuration>
```  

(4) 拷贝整个hadoop安装目录到其他机器

* 启动HDFS  

要运行hadoop的命令，需要在linux环境中配置HADOOP_HOME和PATH环境变量  
```
vi /etc/profile  
export JAVA_HOME=/root/apps/jdk1.8.0_60  
export HADOOP_HOME=/root/apps/hadoop-2.8.1  
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin  
 ```  

初始化namenode的元数据目录：  
要在hdp-01上执行hadoop的一个命令来初始化namenode的元数据存储目录  :  
hadoop namenode -format

启动namenode进程（在node1上):  
hadoop-daemon.sh start namenode    
然后，在windows中用浏览器访问namenode提供的web端口：50070    
然后，启动众datanode们（在任意地方）hadoop-daemon.sh start datanode  
用自动批量启动脚本来启动HDFS  
1)	先配置hdp-01到集群中所有机器（包含自己）的免密登陆  
2)	配完免密后，可以执行一次  ssh 0.0.0.0  
3)	修改hadoop安装目录中/etc/hadoop/slaves（把需要启动datanode进程的节点列入）  
node1     
slave1    
slave2  
slave3 
4)	在node1上用脚本：start-dfs.sh 来自动启动整个集群  
5)	如果要停止，则用脚本：stop-dfs.sh    

1.5 hdfs客户端的常用操作命令  
1. 上传文件到hdfs中

hadoop fs -put /本地文件  /aaa


2. 下载文件到客户端本地磁盘
hadoop fs -get /hdfs中的路径   /本地磁盘目录  

3. 在hdfs中创建文件夹  
hadoop fs -mkdir  -p /aaa/xxx  


4. 移动hdfs中的文件（更名）  
hadoop fs -mv /hdfs的路径1  /hdfs的另一个路径2,复制hdfs中的文件到hdfs的另一个目录  
hadoop fs -cp /hdfs路径_1  /hdfs路径_2  
 

5. 删除hdfs中的文件或文件夹  
hadoop fs -rm -r /aaa  


6. 查看hdfs中的文本文件内容  
hadoop fs -cat /demo.txt  
hadoop fs -tail -f /demo.txt  








 
 

  


  
   
  
