### sqoop 

　*使用Sqoop可以从关系型数据库中导入数据到HDFS上，在这个过程中import操作的输入是一个数据库表，Sqoop会逐行读取记录到HDFS中。import操作的输出是包含读入表的一系列HDFS文件，import操作是并行的也就是说可以启动多个map同时读取数据到HDFS，每一个map对应一个输出文件。这些文件可以是TextFile类型，也可以是Avro类型或者SequenceFile类型。 在import过程中还会生成一个Java类，这个类与输入的表对应，类名即表名，类变量即表字段。import过程会使用到这个Java类。 在import操作之后，就可以使用这些数据来实验export过程了。export是将数据从HDFS或者Hive导出到关系型数据库中。export过程并行的读取HDFS上的文件，将每一条内容转化成一条记录，然后作为一个新行insert到关系型数据库表中。 除了import和export，Sqoop还包含一些其他的操作。比如可以使用sqoop-list-databases工具查询数据库结构，可以使用sqoop-list-tables工具查询表信息。还可以使用sqoop-eval工具执行SQL查询。*


* sqoop安装:
修改配置文件  
```bash
$ cd $SQOOP_HOME/conf  
$ mv sqoop-env-template.sh sqoop-env.sh  
#打开sqoop-env.sh并编辑下面几行：  
export HADOOP_COMMON_HOME=/home/hadoop/apps/hadoop-2.6.1/   
export HADOOP_MAPRED_HOME=/home/hadoop/apps/hadoop-2.6.1/  
export HIVE_HOME=/home/hadoop/apps/hive-1.2.1  
```

* 加入mysql的jdbc驱动包  

```bash
cp  ~/app/hive/lib/mysql-connector-java-5.1.28.jar   $SQOOP_HOME/lib/
```

* 验证启动   
```bash
$ cd $SQOOP_HOME/bin
$ sqoop-version
```

* 下面的语法用于将数据导入HDFS
```bash
$ sqoop import (generic-args) (import-args) 
```



* 测试数据库连接
```bash
bin/sqoop list-databases  --connect jdbc:mysql://hdp20-04:3306/app  --username root  --password root

sqoop create-hive-table --connect jdbc:mysql://hdp20-04:3306/app --table uv_info --username root --password secret_password --hive-table origin_ennenergy_transport.uv_info

sqoop import --connect jdbc:mysql://ip:3306/application  --username root --password secret_password  --table uv_info  --hive-import   --hive-table origin_ennenergy_transport.uv_info  -m 1 --fields-terminated-by "\001";
```

* 将mysql的表导入 hdfs 
```bash
bin/sqoop import \
--connect jdbc:mysql://hdp-04:3306/userdb \
--username root \
--password root \
--target-dir \
/sqooptest \
--fields-terminated-by ',' \
--table emp \
--split-by id \
--m 2
```



* 将mysql的表导入 hive
```bash
bin/sqoop import \
--connect jdbc:mysql://hdp-04:3306/userdb \
--username root \
--password root \
--hive-import \
--fields-terminated-by ',' \
--table emp \
--split-by id \
--m 2
```

* 将mysql的表的增量数据导入 hdfs
```bash
bin/sqoop import \
--connect jdbc:mysql://hdp-04:3306/userdb \
--target-dir /sqooptest  \
--username root \
--password root \
--table emp \
--m 1 \
--incremental append \
--check-column id \
--last-value 1205
```


* 将hdfs的文件数据导出到mysql
```bash
bin/sqoop export \
--connect jdbc:mysql://hdp-04:3306/userdb \
--username root \
--password root \
--input-fields-terminated-by ',' \
--table emp \
--export-dir /sqooptest/
```

* 将hive的表数据（hdfs的文件）导出到mysql
```bash
bin/sqoop export \
--connect jdbc:mysql://hdp-04:3306/userdb \
--username root \
--password root \
--input-fields-terminated-by ',' \
--table t_from_hive \
--export-dir /user/hive/warehouse/t_a/
```



