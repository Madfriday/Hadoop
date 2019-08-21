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


1.3 各种数据库之间的区别：

![diff](iamges/diff.png "diff")
