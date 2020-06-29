## VMware Ubuntu20.04 LST server配置静态网络IP

### 安装时

1. 选择静态IP配置。subnet为xxx.xxx.xxx.0/24，其他的简单

   ```shell
   ipv4
   ip:192.168.153.130
   netgate:192.168.153.1
   ```


2. 设置安装软件、更新源，选择Done,回车

   http://mirrors.aliyun.com/ubuntu

### 安装完成

根据设置的ip地址，修改 虚拟网络编辑器->vmnet8，将子网IP设置为`192.168.153.0` ， 再将网关IP设为`192.168.153.2`

![虚拟机NAT设置](img/虚拟机NAT设置.jpg)



### 复制虚拟机

虚拟机->管理->克隆->创建完整克隆

开启虚拟机，`sudo vim /etc/hostname`，修改主机名

`sudo vim etc/netplan/xx.yaml`，修改IP地址

## Ubuntu相关

### 允许root远程登录

1. `sudo vim /etc/ssh/sshd_config`

    ```shell
    找到 #PermitRootLogin prohibit-password那一行，复制该行内容并修改为
    PermitRootLogin yes
    ```

![允许root远程登录](img/允许root远程登录.jpg)

2. 重启服务：`sudo service sshd restart`

3. 修改密码：`sudo passwd`

### 防火墙

```shell
sudo ufw status
sudo ufw enable | disable
```



### 设置ssh免密登录

1. 修改namenode的hosts文件

    ```
    192.168.153.130 namenode
    192.168.153.131 datanode1
    192.168.153.132 datanode2
    ```

    再修改hostname文件，改成对应的hostname，然后重启

2. 复制该文件给datanode1和datanode2

    `sudo scp /etc/hosts root@192.168.153.131:/etc/hosts `

3. 为namenode、datanode1、datanode2生成密钥对

    `ssh-keygen -t rsa`


4. 登录datanode，将namenode的公钥文件复制给datanode1、datanode2

   `sudo scp /home/skb/.ssh/id_rsa.pub skb@192.168.153.130:/home/skb/.ssh/id_rsa.pub.datanode1`

5. 将三个公钥添加进authorized_keys文件中，并修改文件权限

   ```shell
   cd ~/.ssh
   cat id_rsa.pub >> authorized_keys; cat id_rsa.pub.datanode1 >> authorized_keys; cat id_rsa.pub.datanode2 >> authorized_keys
   chmod 644 authorized_keys # 不改好像也无关系
   ```

6. 将authorized_keys复制给datanode1和datanode2

   `scp ~/.ssh/authorized_keys skb@datanode1:/home/skb/.ssh/authorized_keys`

7. 测试是否可以免密登录

   在namnode节点上输入`ssh datanode1`

## wget

1. 使用wget获取ftp文件：`wget ftp://xx.xx.xx.xx/name.suffix`

## Java环境变量配置

```shell
export JAVA_HOME=/usr/jdk1.8.0_151/
export PATH=${JAVA_HOME}/bin:${PATH}
export CLASS_PATH=${CLASS_PATH}:.:${JAVA_HOME}/lib
```

## Hadoop配置

1. 环境变量配置`.bashrc`

    ```shell
    export HADOOP_INSTALL=/usr/hadoop-3.2.1/
    export PATH=${PATH}:${HADOOP_INSTALL}/bin
    export PATH=${PATH}:${HADOOP_INSTALL}/sbin
    export HADOOP_MAPRED_HOME=${HADOOP_INSTALL}
    export HADOOP_COMMON_HOME=${HADOOP_INSTALL}
    export HADOOP_HDFS_HOME=${HADOOP_INSTALL}
    export HADOOP_YARN_HOME=${HADOOP_INSTALL}
    export HADOOP_CONF_HOME=${HADOOP_INSTALL}
    ```

2. 配置文件

   1. `cd hadoop/ect/hadoop`

   2. 在`hadoop-env.sh`中配置JAVA_HOME，/JAVA_HOME

   3. 在`yarn-env.sh`中配置JAVA_HOME，/JAVA_HOME **暂时没有配置**

   4. 在`workers`中配置ip或hostname

      ```shell
      datanode1
      datanode2
      ```

   5. 配置core-site.xml文件

      ```xml
      <configuration>
              <property>
                      <name>fs.defaultFS</name>
                      <value>hdfs://namenode:9000</value>
              </property>
              <property>
                      <name>hadoop.tmp.dir</name>
                      <value>/home/skb/hd_space/tmp</value>
              </property>
      </configuration>
      ```

   6. 建立hdfs相关目录

      ```shell
      mkdir -p ~/hd_space/tmp; mkdir -p ~/hd_space/hdfs/name; mkdir -p ~/hd_space/hdfs/data; mkdir -p ~/hd_space/mapred/local; mkdir -p ~/hd_space/mapred/system;   
      ```

   7. 配置hdfs-site.xml文件

      ```xml
      <configuration>
              <property>
                      <name>dfs.replication</name>
                      <value>2</value>
              </property>
              <property>
                      <name>dfs.namenode.name.dir</name>
                      <value>file:/home/skb/hd_space/hdfs/name</value>
              </property>
              <property>
                      <name>dfs.datanode.data.dir</name>
                      <value>file:/home/skb/hd_space/hdfs/data</value>
              </property>
              <property>
                      <name>dfs.namenode.http-address</name>
                      <value>namenode:50070</value>
              </property>
              <property>
                      <name>dfs.namenode.secondary.http-address</name>
                      <value>datanode1:50090</value>
              </property>
              <property>
                      <name>dfs.webdfs.enabled</name>
                      <value>true</value>
              </property>
      </configuration>
      ```

   8. 配置mapred-site.xml文件

      ```xml
      <configuration>
              <property>
                      <name>mapred.cluster.local.dir</name>
                      <value>/home/skb/hd_space/mapred/local</value>
              </property>
              <property>
                      <name>mapred.cluster.system.dir</name>
                      <value>/home/skb/hd_space/mapred/system</value>
              </property>
              <property>
                      <name>mapred.framework.name</name>
                      <value>yarn</value>
              </property>
              <property>
                      <name>mapred.jobhistory.address</name>
                      <value>namenode:10020</value>
              </property>
              <property>
                      <name>mapred.webapp.address</name>
                      <value>namenode:19888</value>
              </property>
      </configuration>
      ```

   9. 配置yarn-site.xml文件（该文件配置正确）

      ```xml
      		<property>
                      <name>yarn.resourcemanager.hostname</name>
                      <value>namenode</value>
              </property>
              <property>
                      <name>yarn.nodemanager.aux-services</name>
                      <value>mapreduce_shuffle</value>
              </property>
              <property>
                      <name>yarn.nodemanager.vmem-check-enabled</name>
                      <value>false</value>
              </property>
      
      ```
      
   10. 修改hadoop所属用户与用户组
   
       `sudo chown -R skb:skb hadoop-3.2.1/`

## 建立分布式集群

1. 同步资源及配置文件到数据节点

   ```shell
   for target in datanode1 datanode2
   do
   sudo scp -r /usr/jdk1.8.0_151/ ${target}:/usr
   sudo scp -r /usr/hadoop-3.2.1/ ${target}:/usr
   scp -r /home/skb/hd_space/ ${target}:/home/skb/hd_space
   scp -r /home/skb/.bashrc ${target}:/home/skb/.bashrc
   done
   ```

2. 更改所属用户与用户组

   `sudo chown -R skb:skb hadoop-3.2.1/`

3. 首次启动hadoop，需要格式化hdfs：`hdfs namenode -format`

4. 启动dfs

   ```shell
   cd /usr/hadoop-3.2.1/
   start-dfs.sh
   # 停止命令 stop-dfs.sh
   ```

5. 启动yarn

   `start-yarn.sh / stop-yarn.sh`

6. 也可以使用`start-all.sh`启动hadoop的各个监护进程，`stop-all.sh`关闭各个监护进程

7. 使用`jps`判断hadoop是否配置和启动成功

8. 也可看可视化界面[namenode information](http://192.168.153.130:50070/)、 [Nodes of the cluster](<http://192.168.153.130:8088/>)

## 配置scala

`.bashrc`

```shell
export SCALA_HOME=/usr/scala-2.12.11/
export PATH=${PATH}:${SCALA_HOME}/bin
```

## 配置Spark

1. `vim spark/conf/spark-env.sh.template`

    ```shell
    export SCALA_HOME=/usr/scala-2.12.11
    export JAVA_HOME=/usr/jdk1.8.0_151
    export HADOOP_HOME=/usr/hadoop-3.2.1
    export HADOOP_CONF_HOME=${HADOOP_HOME}/etc/hadoop
    export SPARK_HOME=/usr/spark-3.0.0
    export SPARK_MASTER_IP=192.168.153.130
    ```

    `cp spark-env.sh.template spark-env.sh`

2. 配置slaves

   ```shell
   sudo cp slaves.template slaves
   # 修改slaves
   namenode
   datanode1
   datanode2
   ```

3. 同步配置

   ```shell
   for target in datanode1 datanode2
   do
   sudo scp -r /usr/scala-2.12.11/ ${target}:/usr
   sudo scp -r /usr/spark-3.0.0/ ${target}:/usr
   scp /home/skb/.bashrc ${target}:/home/skb/.bashrc
   done
   ```

4. source一下

5. 修改所属用户与用户组

   `sudo chown -R skb:skb `

6. 启动spark集群

   ```shell
   cd /usr/spark-3.0.0
   ./sbin/start-all.sh
   ./sbin/stop-all.sh
   ```

7. 查看是否启动[Spark Master at spark](<http://192.168.153.130:8080/>)

## SparkSQL ODBC配置

1. 创建Spark工作目录

   `hadoop fs -mkdir /spark-warehouse`

2. 在/usr/spark/conf下添加文件`hive-site.xml` 

   ```xml
   <configuration>
           <property>
                   <name>hive.metastore.warehouse.dir</name>
                   <value>hdfs://namenode:9000/spark-warehouse</value>
           </property>
   </configuration>
   ```

3. 启动jdbc/odbc服务

   `./sbin/start-thriftserver.sh --master spark://namenode:7077`

   `./sbin/stop-thriftserver.sh`

4. 使用beeline进行测试，建表和导入数据

   `./bin/beeline`

5. 连接JDBC

   ```shell
   !connect jdbc:hive2://192.168.153.130:10000
   用户名：skb
   密码：无
   ```

   创建表aqi

   ```sql
   create table aqi(id int, r1 string, timeid string, mon int, day int, hour int, pollutioncode string, revisedflow float)
   row format delimited
   fields terminated by '\t'
   lines terminated by '\n'
   ```

   查看表是否建立：`show tables`

   加载数据：`load data local inpath 'tmp/xx.xx' overwrite into table aqi`，要把文件放入/tmp下

   断开连接：`!q`

