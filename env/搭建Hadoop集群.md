# 搭建Hadoop集群



1. 下载Hadoop3.3.4压缩包，并将该压缩包解压到/opt/目录下

   ```shell
   wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz
   
   tar -zxvf hadoop-3.3.4.tar.gz
   ```

2. 编辑配置文件

   ```shell
   cd /opt/hadoop-3.3.4/etc/hadoop
   ```

   ```
   vim core-site.xml
   ```


- core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>

        <property>
	    <name>fs.defaultFS</name>
	    <value>hdfs://u1:8020</value>
        </property>
		
        <property>
             <name>hadoop.tmp.dir</name>
             <value>/opt/hadoop-3.3.4</value>
        </property>
		
		<property>
	     <name>hadoop.http.staticuser.user</name>
	     <value>hadoop</value>
		</property>
	
		<property>
		<name>hadoop.proxyuser.root.hosts</name>
		<value>*</value>	
		</property>
		
		<property>
		<name>hadoop.proxyuser.root.groups</name>
		<value>*</value>
		</property>
		
</configuration>
```

- hdfs-site.xml

```shell
  vim hdfs-site.xml
```

```xml
<!-- 设定SNN运行主机和端口-->
<configuration>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node2:9868</value>
	</property>
</configuration>
```

- mapred-site.xml

```shell
  vim mapred-site.xml
```

```xml
<configuration>
    <!-- MAPREDUCE程序默认运行方式。yarn集群模式 local本地模式-->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!-- MR APP Master环境变量-->
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <!-- MR MapTask环境变量-->
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <!-- MR Reduce Task环境变量-->
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
</configuration>
```

- yarn-site.xml

```shell
  vim yarn-site.xml
```

```xml
<configuration>
    <!-- yarn集群主角色RM运行机器 -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>u1</value>
    </property>
    <!-- NodeManager上运行的附属服务，需要配置mapreduce_shuffle，才可运行MR程序 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- 每个容器请求的最小内存资源(MB为单位) -->
    <property>
        <name>yarn.scheduler.minimum-allocation-mb</name>
        <value>128</value>
    </property>
    <!-- 每个容器请求的最大内存资源(MB为单位) -->
    <property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>2048</value>
    </property>
    <!-- 容器虚拟内存与物理内存之间的比率 -->
    <property>
        <name>yarn.nodemanager.vmem-pmen-ratio</name>
        <value>4</value>
    </property>
</configuration>
```

- workers

```shell
vim workers  
```

```text
# 配置工人，因为3个节点都是工人，所以写入其在hosts中的缩写即可
u1
u2
u3
```

- hadoop-env.sh

```bash
export JAVA_HOME=/usr/lib/jvm/java
```

将配置好的hadoop拷贝到另外2台机器(如果出错，要检查目的服务器的文件夹权限)



3. 格式化操作，首次启动HDFS时，必须对其进行格式化操作

```bash
hdfs namenode -format
#注意：
#1.只有首次启动需要format操作，后续启动hadoop无需format
#2.如果多次format不仅会造成数据丢失，还会导致hdfs集群主从角色直接互不相识，此时应该删除所有机器下的hadoop.tmp.dir目录，并重新format来解决
```

4. Hadoop集群启动和关闭（启动HDFS集群和YARN集群，这里优先使用start-all.sh来启动)

```shell
方法一：手动逐个进程启停：每台机器上每次手动启动关闭一个角色进程

	HDFS集群启动：
	hdfs --daemon start namenode|datanode|secondarynamenode
	HDFS集群关闭：
	hdfs --daemon stop  namenode|datanode|secondarynamenode
	
	YARN集群启动：
	yarn --daemon start resourcemanager|nodemanager`
	YARN集群关闭：
	yarn --daemon stop resourcemanager|nodemanager

	
先启动HDFS集群
	
	#在主节点u1中执行
	hdfs --daemon start namenode
	hdfs --daemon start datanode
	#在u2中执行
	hdfs --daemon start secondarynamenode
	hdfs --daemon start datanode
	#在u3中执行
	hdfs --daemon start datanode
	#分别在u1~u3中执行以下命令以查看各个角色是否启动成功，否则要查看hadoop日志
	jps
	
再启动YARN集群
	
	#在主节点u1中执行
	yarn --daemon start resourcemanager
	yarn --daemon start nodemanager
	#在从节点u2中执行
	yarn --daemon start nodemanager
	#在从节点u3中执行
	yarn --daemon start nodemanager
	#分别在u1~u3中执行以下命令以查看各个角色是否启动成功，否则要查看hadoop日志
	jps

方法二：shell脚本一键启停：前提是配置好机器之间的SSH免密登陆和workers文件
	
	HDFS集群启动：`start-dfs.sh`
	HDFS集群关闭：`stop-dfs.sh`
	YARN集群启动：`start-yarn.sh`
	YARN集群关闭：`stop-yarn.sh`
	Hadoop集群启动（启动HDFS和YARN这两个集群）：`start-all.sh`
	Hadoop集群关闭（关闭HDFS和YARN这两个集群）：`stop-all.sh`
	具体执行过程，只需在node1，即主节点执行：

	start-all.sh
	stop-all.sh
	

Shell中测试HDFS的文件上传功能：

	#在HDFS集群中生成目录/demo1
	hadoop fs -mkdir /demo1
	#在服务器node1中生成简单的测试文件
	echo a >> a_simple_word.txt
	#将文件上传到HDFS集群的/demo1目录下
	hadoop fs -put a_simple_word.txt /demo1
	#查看HDFS根目录下的文件夹
	hadoop fs -ls /
```







