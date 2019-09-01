### 自定义分组，分区

* 数据如下

|orderId|userId|product|price|number|
|------|-------|-------|-----|------|
|order001|u001|小米6|1999.9|2|
|order001|u001|雀巢咖啡|99.0|2|
|order001|u001|安慕希|250.0|2|
|order001|u001|经典红双喜|200.0|4|
|order001|u001|防水电脑包|400.0|2|
|order002|u002|小米手环|199.0|3|
|order002|u002|榴莲|15.0|10|
|order002|u002|苹果|4.5|20|
|order002|u002|肥皂|10.0|40|
|order003|u001|小米6|1999.9|2|
|order003|u001|雀巢咖啡|99.0|2|
|order003|u001|安慕希|250.0|2|
|order003|u001|经典红双喜|200.0|4|
|order003|u001|防水电脑包|400.0|2|  



* 统计要求：reduce按照orderId分组读取map产生的数据，reduce结果按orderId分区写入，求出总价sumprice最大的前3.   

* 解决过程：

1.1 定义一个orderbeans用于存放一条数据中的所有字段，并实现序列化与比较器。

```java
public class OrderBeans implements WritableComparable<OrderBeans>{
	
	private String orderId;
	private String userId;
	private String product;
	private float price;
	private int number;
	//创建sum使orderbens用总价来排序。
	private float sum;
	
	
	public OrderBeans() {
		
	}
	
	public void set(String orderId, String userId, String product, float price, int number) {
		this.orderId = orderId;
		this.userId = userId;
		this.product = product;
		this.price = price;
		this.number = number;
		this.sum = this.price * this.number;
	}
		

	public String getOrderId() {
		return orderId;
	}

	public void setOrderId(String orderId) {
		this.orderId = orderId;
	}

	public String getUserId() {
		return userId;
	}

	public void setUserId(String userId) {
		this.userId = userId;
	}

	public String getProduct() {
		return product;
	}

	public void setProduct(String product) {
		this.product = product;
	}

	public double getPrice() {
		return price;
	}

	public void setPrice(float price) {
		this.price = price;
	}

	public int getNumber() {
		return number;
	}

	public void setNumber(int number) {
		this.number = number;
	}

	public float getSum() {
		return sum;
	}

	public void setSum(float sum) {
		this.sum = sum;
	}

	@Override
	public void readFields(DataInput in) throws IOException {
		this.orderId = in.readUTF();
		this.userId = in.readUTF();
		this.product = in.readUTF();
		this.price = in.readFloat();
		this.number = in.readInt();
		this.sum = this.price * this.number;
		
	}

	@Override
	public void write(DataOutput out) throws IOException {
		out.writeUTF(orderId);
		out.writeUTF(userId);
		out.writeUTF(product);
		out.writeDouble(price);
		out.writeInt(number);
	}

	@Override
	public int compareTo(OrderBeans o) {
		//利用float的方法使float类型的sum可以比较
		return Float.compare(this.sum, o.getSum()) == 0 ? o.getOrderId().compareTo(this.orderId) : Float.compare(this.sum, o.getSum());
	}

}
```

1.2 自定义分组器，按照orderid分组

```java
public class OrderGrouping extends WritableComparator{
	
	public OrderGrouping() {
	super(OrderBeans.class,true);
	}
	
	@Override
	public int compare(Object a,Object b) {
		
		OrderBean a1 = (OrderBean)a;
		OrderBean b1 = (OrderBean)b;
		
		return a1.getOrderId().compareTo(b1.getOrderId());
	}
}
```

1.3 自定义分区器，按照orderid分区。

```java
public class orderPartition extends Partitioner<OrderBeans, NullWritable>{

	@Override
	public int getPartition(OrderBeans arg0, NullWritable arg1, int arg2) {
		
		return arg0.getOrderId().hashCode() & Integer.MAX_VALUE % arg2;
	}
	

}
```

1.4 创建mapreduce，统计出总价前3 的数据。

```java
public class OrderTopn {
	private static class map extends Mapper<LongWritable, Text, OrderBeans, NullWritable>{
		@Override
		protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, OrderBeans, NullWritable>.Context context)
				throws IOException, InterruptedException {
			String[] values = value.toString().split(",");
			OrderBeans ordersbeans  = new OrderBeans();
			ordersbeans.set(values[0], values[1], values[2], Float.parseFloat(values[3]), Integer.parseInt(values[4]));
			context.write(ordersbeans, NullWritable.get());
		}
	}

	private static class reduce extends Reducer<OrderBeans, NullWritable, OrderBeans, NullWritable>{
		@Override
		protected void reduce(OrderBeans arg0, Iterable<NullWritable> arg1,
				Reducer<OrderBeans, NullWritable, OrderBeans, NullWritable>.Context arg2)
				throws IOException, InterruptedException {
			int  i = 0;
			//将sum前三的写入context
			for(NullWritable nulls : arg1) {
				arg2.write(arg0, nulls);
				i++;
				if(i <= 2) {
					return;
				}
			}
		}
	}

	public static void main(String[] args) throws Exception, IOException {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJar(args[0]);
		//创建jar包路径
		job.setMapperClass(map.class);
		job.setReducerClass(reduce.class);
		job.setMapOutputKeyClass(OrderBean.class);
		job.setMapOutputValueClass(NullWritable.class);
		job.setOutputKeyClass(OrderBean.class);
		job.setOutputValueClass(NullWritable.class);
		job.setPartitionerClass(Orderpat.class);
		job.setGroupingComparatorClass(OrderGroping.class);
		FileInputFormat.setInputPaths(job, new Path(args[1]));
		File p1 = new File(args[2]);
		if(p1.exists()) {
			p1.delete();
		}
		FileOutputFormat.setOutputPath(job, new Path(p1.getAbsolutePath()));
		job.waitForCompletion(true);
	}
}
```
