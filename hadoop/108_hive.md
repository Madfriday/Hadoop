###hive install & use

1.1 mysql安装：  


* 上传mysql安装包  
* 解压：tar -xvf MySQL-5.6.26-1.linux_glibc2.5.x86_64.rpm-bundle.tar   
* 安装mysql的server包,rpm -ivh MySQL-server-5.6.26-1.linux_glibc2.5.x86_64.rpm   
* 依赖报错：缺perl,yum install perl,安装完perl后 ，继续重新安装mysql-server

```
[root@mylove ~]# rpm -ivh MySQL-server-5.6.26-1.linux_glibc2.5.x86_64.rpm   
又出错：包冲突conflict with  
移除老版本的冲突包：mysql-libs-5.1.73-3.el6_5.x86_64
[root@mylove ~]# rpm -e mysql-libs-5.1.73-3.el6_5.x86_64 --nodeps  
   
继续重新安装mysql-server  
[root@mylove ~]# rpm -ivh MySQL-server-5.6.26-1.linux_glibc2.5.x86_64.rpm   

成功后，注意提示：里面有初始密码及如何改密码的信息  
初始密码：/root/.mysql_secret    
改密码脚本：/usr/bin/mysql_secure_installation  

安装mysql的客户端包：  
[root@mylove ~]# rpm -ivh MySQL-client-5.6.26-1.linux_glibc2.5.x86_64.rpm  

启动mysql的服务端：  
[root@mylove ~]# service mysql start  
Starting MySQL. SUCCESS!  
 

修改root的初始密码：  
[root@mylove ~]# /usr/bin/mysql_secure_installation  按提示  

测试：  
用mysql命令行客户端登陆mysql服务器看能否成功  
[root@mylove ~]# mysql -uroot -proot  
mysql> show databases;  
```

1.2  hive的元数据库配置

* vi conf/hive-site.xml:

```
<configuration>
<property>
<name>javax.jdo.option.ConnectionURL</name>
<value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
<description>JDBC connect string for a JDBC metastore</description>
</property>

<property>
<name>javax.jdo.option.ConnectionDriverName</name>
<value>com.mysql.jdbc.Driver</value>
<description>Driver class name for a JDBC metastore</description>
</property>

<property>
<name>javax.jdo.option.ConnectionUserName</name>
<value>root</value>
<description>username to use against metastore database</description>
</property>

<property>
<name>javax.jdo.option.ConnectionPassword</name>
<value>root</value>
<description>password to use against metastore database</description>
</property>
</configuration>
```

* 上传一个mysql的驱动jar包到hive的安装目录的lib中  
* 配置HADOOP_HOME 和HIVE_HOME到系统环境变量中：/etc/profile  
* source /etc/profile,hive启动测试


1.3 hive的操作：

外部表(EXTERNAL_TABLE)：表目录由建表用户自己指定 

```
create external table t_access(ip string,url string,access_time string)  
row format delimited  
fields terminated by ','  
location '/access/log';  
```

*外部表和内部表的特性差别：  
1、内部表的目录在hive的仓库目录中 VS 外部表的目录由用户指定  
2、drop一个内部表时：hive会清除相关元数据，并删除表数据目录  
3、drop一个外部表时：hive只会清除相关元数据*

带分区的表
```
create table t_access(ip string,url string,access_time string)
partitioned by(dt string)
row format delimited
fields terminated by ',';
```


1.4 hive基本思想：  
Hive是基于Hadoop的一个数据仓库工具(离线)，可以将结构化的数据文件映射为一张数据库表，并提供类SQL查询功能。

![HIVE](images/hive.png "hive")

* hive特点：   

可扩展   
Hive可以自由的扩展集群的规模，一般情况下不需要重启服务。  

延展性   
Hive支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。  
 
容错   
良好的容错性，节点出现问题SQL仍可完成执行。  



* hive 操作案例

1.1 

```sql
create table t_101(name String,number int)     
row formate delimited     
fields terminated by ',';  
```
数据如下:  
a,1  
b,1  
c,1  

```sql
create table t_102(name String,number int)     
row formate delimited     
fields terminated by ',';  
```
数据如下:  
a,21  
b,11  
c,31  

要求：将/root/t1.sh,t2.sh分别放入这两个表中，实现内连接，左连接，全外连接
```sql
load data local inpath '/root/t1.sh' into table t_101;
load data local inpath '/root/t2.sh' into table t_102;

内连接  
select a.*, b.* from t_101 a join t_102 b;  
内连接为笛卡尔积，左边一个连对面所有；  
 左连接  
select a.*, b.* from t_101 a left join t_102 b on a.name = b.name;  
全外连接  
select a.*, b.* from t_101 a full outer join t_102 b on a.name = b.name;  
```
  
  
1.2

(聚合与分组) 

```sql
create table t_103(ip String,url String,time String)     
row formate delimited     
fields terminated by ',';  
```
数据如下：  
192.163.33.4,http://sina.com/f,2017-09-17  
192.163.33.2,http://sina.com/b,2017-09-13  
192.163.33.6,http://sina.com/h,2017-09-19  
192.163.33.9,http://sina.com/c,2017-09-15  
192.163.33.0,http://sina.com/s,2017-09-13  
192.163.33.2,http://sina.com/f,2017-09-13  
192.163.33.4,http://sina.com/g,2017-09-14  
192.163.33.8,http://sina.com/p,2017-09-15  
192.163.32.1,http://sina.com/p,2017-09-16  
192.163.31.0,http://sina.com/i,2017-09-17  
192.163.36.7,http://sina.com/fv2017-09-18  
192.163.32.2,http://sina.com/b,2017-09-19  
192.163.34.6,http://sina.com/n,2017-09-11   
192.163.36.6,http://sina.com/m,2017-09-12  
```
load data local inpath '/root/t1.sh' into table t_103;
```
要求：1，求出ip，url（转为大写）；2，统计url的数量；3，求出每个url个数以及ip最大的；4，求每个用户ip访问的数量，访问页面以及数量以及时间最晚的。  

```sql
select ip,upper(url) from t_103;
select url,count(1) as counts from t_103 group by url;
select url,max(ip) from t_103 group by url;
seelect ip,count(1) as count_ip,url,count(1) s count_url,max(time) from t_103 group by ip,url;
```

1.3 
```sql
create table t_104(ip String,url String, day String)
partitioned by (time String)
row fromat delimited
fields terminated by ',';
192.168.33.1,thhp://www.cn/stu,2017-08-02  
192.168.33.2,thhp://www.cn/cnbc,2017-08-02  
192.168.33.1,thhp://www.cn/st21,2017-08-02  
192.168.33.3,thhp://www.cn/stas,2017-08-02  
192.168.33.6,thhp://www.cn/cnbc,2017-08-02  
192.168.33.8,thhp://www.cn/sst21,2017-08-02  
192.168.33.9,thhp://www.cn/stu,2017-08-02  
192.168.33.0,thhp://www.cn/aka,2017-08-02  
192.168.33.2,thhp://www.cn/aka,2017-08-02  
192.168.33.5,thhp://www.cn/stu,2017-08-02  

192.168.33.1,thhp://www.cn/stu,2017-08-03  
192.168.33.3,thhp://www.cn/cnbc,2017-08-03  
192.168.33.4,thhp://www.cn/st21,2017-08-03   
192.168.33.3,thhp://www.cn/stas,2017-08-03    
192.168.33.6,thhp://www.cn/cnbc,2017-08-03  
192.168.33.6,thhp://www.cn/sst21,2017-08-03  
192.168.33.1,thhp://www.cn/stu,2017-08-03  
192.168.33.0,thhp://www.cn/aka,2017-08-03  
192.168.33.8,thhp://www.cn/aka,2017-08-03  
192.168.32.5,thhp://www.cn/stu,2017-08-03  

192.168.33.1,thhp://www.cn/stu,2017-08-04  
192.168.34.2,thhp://www.cn/cnbc,2017-08-04   
192.168.36.1,thhp://www.cn/st21,2017-08-04   
192.168.33.3,thhp://www.cn/stas,2017-08-04   
192.168.33.6,thhp://www.cn/cnbc,2017-08-04   
192.168.32.8,thhp://www.cn/sst21,2017-08-04   
192.168.31.9,thhp://www.cn/stu,2017-08-04   
192.168.39.0,thhp://www.cn/aka,2017-08-04   
192.168.37.2,thhp://www.cn/aka,2017-08-04   
192.168.30.5,thhp://www.cn/cnbc,2017-08-04  

load data inpath '/root/t1.sh into table t_104 partition (time ='08-03)';  
load data inpath '/root/t2.sh into table t_104 partition (time ='08-04)';  
load data inpath '/root/t3.sh into table t_104 partition (time ='08-05)';  
```

要求：1，求8.2以后每一天http；//www.cn.stu 这条网页访问次数以及ip最大者；  
2.求每天每一个页面访问总次数以及ip最大者;  
3.求上述页面总访问次数小于2次的(子查询);

```sql
select time,count(1) as count)url,max(ip) from t_104 where url = 'http://www.cn/stu' and time > '2017-08-02' group by time,url;  
select time,url,count(1) as count_url,max(ip) from t_104 group by time,url;  
select url,count_url from (select time,url,count(1) as count_url,max(ip) from t_104 group by time,url) tmp where tmp.count_url > 2;  
```

1.4 

```sql
create table t_105(movie_name String,actors array<String>,first_show String)
row fromat delimited
fields terminated by ','
collection items terminated by ':';
```

数据如下:  
肖申克德救赎，史提夫:约翰逊:艾比利，1994-03-14   
卧虎藏龙，周润发：杨紫琼：李安：王宝强，2002-01-21   
蜘蛛侠，汤姆赫兰德：赞达亚杰克：吉伦哈尔，2019-06-28   

要求：1，选择电影，每个电影第一个演员及首映。    
2.选择电影及演员人数。   
3.选择电影及演员中包括王宝强的。  
4.选择电影，演员，首映，演员数量。   
5.查询电影，首映，演员如果包含王宝强定义为喜剧片。  

```sql
select movie_name,actors[0], first_show from t_105;
select movie_name,size(actors) from t_105;
select movie_name,actors from t_105 where array_contains(actors,'王宝强');
select movie_name,actors,first_show,size(actors) as count_actors from t_105;
select movie_name,first_show,actors,if(array_contains(actors,'王宝强'),'喜剧片','动作片');
```

1.5
数据如下:  
1.zhang,father:xiao#mother:li#brother:xiaoo,28  
2.feng,father:liu#mother:weiw#sister:xil,28  
3.zeng,father:wei#mother:ip#brother:xis,22  
4.zhan,father:he#mother:op#sister:xia,21  
5.chang,father:xie#mother:op#brother:xic,23  
6.hang,father:feng#mother:pop#sister:xij,24  
 
```sql
create table t_106 (id int,name String,family map<String,String>)  
row fromat delimited  
fields terminated by ','  
collection items terminated by ':'  
map keys terminated by ':';  
load data local  inpath '/root/t1.sh' into table t_106;
```
要求：1选择id，姓名，家庭中父亲
2.选择id，姓名，所有家庭亲属关系
3.选择id，name，家庭成员数量
4.查询拥有兄弟人的id，姓名

```sql
select id,name,family['father'] from  t_106
select id,name,map_keys(family) as relation from t_106
select id,name,size(family) from t_106
select id,name from t_106 where array_contains(map_keys(family),'brother');
```

1.6
数据如下:  
1,zhang,18:male:beijing  
2,li,11:famale:shanghai  
3,liu,12:male:tianjing  
4,xie:16:famale:changdu  
5,yaun,13:famale:chengdu  
6,xue,23:famale:shengzheng  
7,sun,61:male:xinjia  

```sql
create table t_107(id int,name String,info struct<age:int,sex;String,add:String>)
row fromat delimited
fields terminated by ','
collection items terminated by ':';
```

要求：1查询id，姓名，地址
```sql
select  id,name,info.add from t_107;
```

1.7  
数据如下：   
1.zhasng,a:b:c:f  
2,li,b:d:a:c  
3,wang,d:a:c:e 


```sql
create table t_108(id int,name String,sex array<Stirng>)
row fromat delimited
fields terminated by ','
collection items terminated by ':';
去掉sex中重复的，单独的个数
select distinct tmp.sub from (select explode(sex) as sub from t_108) as tmp;
```

1.8
数据如下:  
hello tom hello jim    
hello rose hello tom    
tom love rose rose love jim  
jim love tom love is what  
what is love  
  
要求：统计每个词的个数

```sql
create table t_109(sentence String)
load data local inpath '/root/t1.sh' into t_109;
select word,count(1) from (select explode(sentence,' ') as word from t_109) tmp group by word;
```

1.9
```sql
create table t_110(id int,name String,age int)
row fromat delimited
fields terminated by ',';
```
数据如下:  
1,zhang,18  
2,li,21  
3,liu,32  
4,xie,46  
5,yuan,13  
6,xue,33  
7,sun,61  
8,liuu,53  
9,yangy,22  

要求：查询id，姓名，年龄且年龄<20为yong，20<age<30为mid，age>30为old
```sql
select id,name,case when age<20 then 'yong',
when 20 < age < 30 then 'mid' else 'old'
from t_110 end;
```

2.0 

```sql 
create table t_111(id int,age int,name String,sex String)
row fromat delimited
fields terminated by ',';
```

数据如下:  
1,18,a,male  
2,19,b,male  
3,22,c,female  
4,16,d,female  
5,30,e,male  
6,26,f,male  
7,36,w,female  
8,15,x,male  
9,20,z,female  
10,31,v,male  

要求：求出每种性别年龄最大的两个人的id，name，sex；  

```sql
select * from (select id,name,sex,row_number() over(partition by sex order by age desc) as rn from t_111) tmp where rn <= 2;
```