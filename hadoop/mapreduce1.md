###Mapreduce工作原理

* 分布式计算原理：
  * map阶段：将hdfs中的block(128M)分给maptask，maptask按照每一行读取文本，将每一行文本按相关规则切割，形成 一组(key => value)的数据  
  * reuduce阶段：定义reducetask个数，将maptask产生的数据，kv值，按照key值相等的kv分配给同一个reduce运算，当同一key在不同的机器上时，需要shuffle通过网络将不同机器上的同一kv值传递给同一reducetask，而后reducetask将具有相同key值的kv按一定规则聚合，最后写入磁盘。  
  * 任务调度(Yran):集群中有唯一resoucemanager，很多台nodemanager，yarn负责调度管理资源。
  
 * mapreduce工作示意图如下：   
 
 ![mapreduce](images/121.png "mapreduce")
  
  
