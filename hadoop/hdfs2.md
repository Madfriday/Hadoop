###HDFS 核心工作原理namenode元数据管理要点  


**namenode元数据管理**  

1、hdfs的目录结构及每一个文件的块信息(块的id，块的副本数量，块的存放位置<datanode>)就是元数据

2、元数据由namenode负责管理   


3、namenode把元数据记录在:      
* namenode的实时的完整的元数据存储在内存中；  
* namenode还会在磁盘中（dfs.namenode.name.dir）存储内存元数据在某个时间点上的镜像文件；  
* namenode会把引起元数据变化的客户端操作记录在edits日志文件中；  
* secondarynamenode会定期从namenode上下载fsimage镜像和新生成的edits日志，然后加载fsimage镜像到内存中，然后顺序解析edits文件，对内存中的元数据对象进行修改(整合),整合完成后，将内存元数据序列化成一个新的fsimage，并将这个fsimage镜像文件上传给namenode  
* 上述过程叫做：checkpoint操作，第一次secondarynamenode需要从namenode中下载fsimage文件，以后就不用了，只需析edits即可  


4、原理如下图所示  
![原理图](images/101.png "原理图")  





