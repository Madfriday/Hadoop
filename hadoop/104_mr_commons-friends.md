### finds commons friends

* 数据如下：

---

A:B,C,D,F,E,O  
B:A,C,E,K  
C:F,A,D,I  
D:A,E,F,L  
E:B,C,D,M,L  
F:A,B,C,D,E,O,M  
G:A,C,D,E,F  
H:A,C,D,E,O  
I:A,O  
J:B,O  
K:A,C,D  
L:D,E,F  
M:E,F,G  
O:A,H,I,J  

---

* 统计要求：找出A-O的所有共同好友  

* 解决过程：

1.1 在map阶段，将每一行数据分key为@的好友，value为@，将其写入context传给reduce；reduce读到的数据为一个好友key以及他的所有value对象，而value值进行两两组合，就是value1，value2 有共同好友key。用一个arraylist保存这些value，再排序，从A到O，将其作为key，将原来的kery作为value写入context。

```java
public class FriendsCount {
	private static class map extends Mapper<LongWritable, Text, Text, Text>{
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, Text>.Context context)
				throws IOException, InterruptedException {
			//分割原始数据，分别获得user以及他的friends
			String[] values = value.toString().split(":");
			String user = values[0];
			String[] friends = values[1].split(",");
			//将friend与user写入context；
			for(String fr : friends) {
				context.write(new Text(fr), new Text(user));
			}
			
		}
	}

	private static class reduce extends Reducer<Text, Text,Text, Text>{
		@Override
		protected void reduce(Text arg0, Iterable<Text> arg1, Reducer<Text, Text, Text, Text>.Context arg2)
				throws IOException, InterruptedException {
			//创建一个arraylist储存user；
			ArrayList<String> user = new ArrayList<String>();
			for(Text friends : arg1) {
				user.add(friends.toString());
			}
			//对arraylist排序
			Collections.sort(user);
			//将有共同好友arg0的user两两组合
			for(int i =0; i < user.size() -1;i ++) {
				for(int j =0; j< user.size(); j++) {
					arg2.write(new Text(user.get(i)+"-"+ user.get(j)), arg0);
				}
			}
			
		}
	}
	
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		//获取jar包
		job.setJar(args[0]);
		job.setMapperClass(map.class);
		job.setReducerClass(reduce.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		//创建读写路径
		FileInputFormat.setInputPaths(job, new Path(args[1]));
		FileOutputFormat.setOutputPath(job, new Path(args[2]));
		boolean b = job.waitForCompletion(true);
		System.exit(b ? 1: 0);
	}

}
```

2.2 上一步mapreduce的结果为不同两两user组合具有共同好友的数据，需要进一步计算，将同一两两user组合数据写入同一文件。需要再创建一个mapreduce，map阶段读取的数是相同的suer组合数据key，把数据按”\t“切割，写入context。reduce阶段将value值按字符串拼接，再写入context中。

```java
public class FriendsCountA2 {
	private static class map extends Mapper<LongWritable, Text, Text, Text>{
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, Text>.Context context)
				throws IOException, InterruptedException {
			String[] values = value.toString().split("\t");
			String couples = values[0];
			String friend = values[1];
			//将一对user的couple，共同好友friend写入context
			context.write(new Text(couples), new Text(friend));
		}
	}

	private static class reduce extends Reducer<Text, Text, Text, Text>{
		@Override
		protected void reduce(Text arg0, Iterable<Text> arg1, Reducer<Text, Text, Text, Text>.Context arg2)
				throws IOException, InterruptedException {
			 String words = "";
			 //将共同好友按字符串拼接
			 for(Text word : arg1) {
				 words += word.toString() + "->";
			 }
			 arg2.write(arg0, new Text(words));
		}
	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		//创建jar包读取路径
		job.setJar(args[0]);
		job.setMapperClass(map.class);
		job.setReducerClass(reduce.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		//创建读写路径
		FileInputFormat.setInputPaths(job, new Path(args[1]));
		FileOutputFormat.setOutputPath(job, new Path(args[2]));
		boolean b = job.waitForCompletion(true);
		System.exit(b ? 1: 0);
	}
}
```
