# hadoop应用程序编写与运行

hadoop版本3.2.1

## maven依赖

```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>3.2.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-client -->
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>3.2.1</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs -->
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-hdfs</artifactId>
    <version>3.2.1</version>
</dependency>
```

## maven的build配置文件

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <archive>
                    <manifest> <!-- 这里需要修改应用程序的入口类 -->
                        <mainClass>com.sonkabin.mr.WordCountDriver</mainClass>
                    </manifest>
                </archive>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 编写Mapper

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

// KEYIN,VALUEIN,KEYOUT,VALUEOUT
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
    // 输入文件的形式：skb a skb
    // 				  ...
    // 输出形式：skb 1 
    //    			  a 1 
    //				  skb 1
    Text k = new Text();
    IntWritable v = new IntWritable(1); // 每个单词先都记为1

    @Override
    protected void map(LongWritable key, Text value, Context context) throws Exception {
        String line = value.toString(); // String和Hadoop中的Text类相对应
        String[] words = line.split(" ");
        for(String word : words){
            k.set(word);
            context.write(k, v);
        }
    }
}
```

## 编写Reducer

```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

// KEYIN,VALUEIN,KEYOUT,VALUEOUT：这里的输入键、值必须和map的输出键、值相同
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
    
    // 输出形式：skb 2
    //		   	a 1
    IntWritable v = new IntWritable();
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws Exception {
        
        int sum = 0;
        for(IntWritable v : values){
            sum += v.get();
        }
        v.set(sum); // reduce值

        context.write(key, v);
    }
}
```

## 编写Driver

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCountDriver {

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        // 获取Job对象
        Job job = Job.getInstance(conf);

        // 设置jar存储位置
        job.setJarByClass(WordCountDriver.class);        

        // 关联Map和Reduce类
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);

        // 设置Mapper阶段输出数据的key和value类型
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // 设置最终数据输出的key和value类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // 设置输入路径和输出路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        // 提交Job
        boolean res = job.waitForCompletion(true);
        System.exit(res ? 0 : 1);
    }
}
```



## 打jar包

`mvn install`

## 上传到hadoop根目录下

## 将需要的文件复制到hdfs的input目录中

`hadoop fs -put 本地文件 /user/skb/input`

## 执行MapReduce

`hadoop jar xxx.jar 入口类全类名 /user/skb/input /usr/skb/output`

## 补充

1. 利用hdfs Client可以直接操作hdfs

   ```java
   import java.net.URI;
   
   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;
   
   public class HDFSClient {
       public static void main(String[] args) throws Exception {
           Configuration conf = new Configuration();
           // 获取hdfs客户端对象
           FileSystem fs = FileSystem.get(new URI("hdfs://192.168.153.130:9000"), conf, "skb");
   
           // 在hdfs上创建路径
           fs.mkdirs(new Path("/user/skb/input"));
   
           // 关闭资源
           fs.close();
   
           System.out.println("over");
       }
   }
   ```

   

2. 