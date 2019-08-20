###MR自定义分区器

* 数据类型：
* 数据形式



|电话|座机|身份|网址|网站|时间|日期|访问下限|访问上限|费用|  
|-----|-----|-----|-----|-----|-----|-----|-----|-----|  
|1363157985066|13726230503|00-FD-07-A4-72-B8:CMCC|120.196.100.82|i02.c.aliimg.com|24|27|2481|24681|200|  
|-|-|-|-|-|-|-|-|-|-|




*统计要求:  
按照电话号码划分归属地，并将相同归属地的电话的数据分为同一区，统计 每一电话的所有上行流量，下行流量。

* 解决过程：

1.1 先创建一个FLowbeans用于存放用于存放一条数据中的上行流量，下行流量，并将其序列化。  


```java
/**
 * created by madfriday
 * @author Administrator
 *
 */

public class Flowbeans implements Writable{
	/**
	 * 创建一个Flowbeans类，用于存放一条数据中的上行流量，下行流量，并采用序列化。
	 */
	//upflow，dflow分别用于存放上行流量，下行流量
	private int upflow;
	private int dflow;
	
	//构造一个空参构造器
	public Flowbeans() {
		
	}

	public Flowbeans(int upflow, int dflow) {
		super();
		this.upflow = upflow;
		this.dflow = dflow;
	}
	
	public int getUpflow() {
		return upflow;
	}
	
	public void setUpflow(int upflow) {
		this.upflow = upflow;
	}
	
	public int getDflow() {
		return dflow;
	}
	
	public void setDflow(int dflow) {
		this.dflow = dflow;
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.upflow = in.readInt();
		this.dflow = in.readInt();
		
	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeInt(upflow);
		out.writeInt(dflow);
		
	}
	

}
```

---
1.2 创建一个分区器provincepartitioner,继承Partitioner,创建一个hashmap用于存储电话前三位以及对应的分区。重写方法getPartition，将分区结果返回。


```java
//创建一个分区器继承partitioner
public class ProvincePartitoner extends Partitioner<Text, Flowbeans>{
	//创建一个hashmap将电话前三位作key，对应分区数作为value存放其中。
	static HashMap<String, Integer> map = new HashMap<String, Integer>();
	
	static {
		map.put("135", 0);
		map.put("136", 1);
		map.put("137", 2);
		map.put("138", 3);
	}

	@Override
	public int getPartition(Text arg0, Flowbeans arg1, int arg2) {
		//从map传到reduce的结果为(PhoneNumber,Flowbeans),将电话号码前三位截取出来，通过hashmap取得对应的分区数。
		String keyString = arg0.toString().substring(0,3);
		
		Integer partitionNum = map.get(keyString);
		//将分区数返回
		return partitionNum == null ? 4 : partitionNum;
	}
	

}
``` 


---
1.3 创建mapreduce类，读取数据，将其按照规则切割，最终写入hdfs中。

```java
public class MapReduceJob {
	private static class map extends Mapper<LongWritable, Text, Text, Flowbeans>{
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, Flowbeans>.Context context)
				throws IOException, InterruptedException {
			//分割传入的一行数据，拿到需要的字段，写入context里。
			String[] content = value.toString().split("\t");
			
			String phoneNumber = content[1];
			
			int upflow = Integer.parseInt(content[content.length - 3]);
			
			int dflow = Integer.parseInt(content[content.length - 2]);
			
			context.write(new Text(phoneNumber), new Flowbeans(upflow, dflow));
			
		}
	}
	
	private static class reduce extends Reducer<Text, Flowbeans, Text, Flowbeans>{
		@Override
		protected void reduce(Text arg0, Iterable<Flowbeans> arg1,
				Reducer<Text, Flowbeans, Text, Flowbeans>.Context arg2) throws IOException, InterruptedException {
			int upflow = 0;
			int dflow = 0;
			for(Flowbeans fb : arg1) {
				upflow += fb.getUpflow();
				dflow += fb.getDflow();
			}
			//通过迭代器拿到需要的上行流量，下行流量，求和，写入context。
			arg2.write(arg0, new Flowbeans(upflow, dflow));
		}
	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		//创建job
		Job job = Job.getInstance(conf);
		//通过job设置所需参数，并指定分区器的类
		job.setMapperClass(map.class);
		job.setReducerClass(reduce.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Flowbeans.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Flowbeans.class);
		job.setPartitionerClass(ProvincePartitoner.class);
		//设置数据读入。写出位置。
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		boolean b = job.waitForCompletion(true);
		System.exit(b ?1 : 0);
				
	}

}
```