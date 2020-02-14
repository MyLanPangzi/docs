# Hadoop

## 单机-伪分布式环境搭建

**CentOS 8**

### Steps

1. 下载jdk

   ```shell
   #https://jdk.java.net/
   wget --no-check-certificate --no-cookies \
   -O /opt/software/openjdk-13.0.2.tar.gz \ 
   https://download.java.net/java/GA/jdk13.0.2/d4173c853231432d94f001e99d882ca7/8/GPL/openjdk-13.0.2_linux-x64_bin.tar.gz \ 
   && tar -zxvf /opt/software/openjdk-13.0.2.tar.gz -C /opt/module/
   ```

2. 下载hadoop

   ```shell
   #https://hadoop.apache.org/releases.html
   #http://mirror.bit.edu.cn/apache/hadoop/core/
   wget --no-check-certificate --no-cookies \
   -O /opt/software/hadoop-2.10.0.tar.gz \
   https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/core/hadoop-2.10.0/hadoop-2.10.0.tar.gz \
   && tar -zxvf /opt/software/hadoop-2.10.0.tar.gz -C /opt/module/
   
   ```

3. 配置环境变量

   ```shell
   echo '
   export HADOOP_SSH_OPTS="-p 2222"
   export HADOOP_HOME=/opt/module/hadoop-2.10.0
   export JAVA_HOME=/opt/module/jdk-13.0.2
   export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   ' | sudo tee -a /etc/profile
   
   source /etc/profile
   
   ```

4. 运行单节点案例：grep，wordcount

   ```shell
   cd /opt/module/hadoop-2.10.0 && mkdir input && cp etc/hadoop/*.xml input
   bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar grep input output 'dfs[a-z.]+'
   cat output/*
   
   mkdir wcinput && tee wcinput/wc.input <<-EOF
   hadoop yarn
   hadoop mapreduce
   atguigu
   atguigu
   EOF
   bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount wcinput wcoutput 
   cat wcoutput/*
   
   ```

   

5. 配置配置伪分布式环境

   ```shell
   #先修改etc/hadoop/hadoop-ev.sh的JAVA_HOME改为自己的jdk目录，否则会报错
   cp etc/hadoop/core-site.xml etc/hadoop/core-site.xml.bak
   tee etc/hadoop/core-site.xml <<-EOF
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   
   <configuration>
   <!-- 指定HDFS中NameNode的地址 -->
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://hadoop100:9000</value>
       </property>
   
   <!-- 指定Hadoop运行时产生文件的存储目录 -->
       <property>
           <name>hadoop.tmp.dir</name>
           <value>/opt/module/hadoop-2.10.0/data/tmp</value>
       </property>
   </configuration>
   EOF
   
   cp etc/hadoop/hdfs-site.xml etc/hadoop/hdfs-site.xml.xml.bak
   tee etc/hadoop/hdfs-site.xml <<-EOF
   <?xml version="1.0" encoding="UTF-8"?>
   <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
   
   <configuration>
       <!-- 指定HDFS副本的数量 -->
       <property>
           <name>dfs.replication</name>
           <value>1</value>
       </property>
   </configuration>
   EOF
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   chmod 0600 ~/.ssh/authorized_keys
   ssh -p 2222 localhost
   exit
   bin/hdfs namenode -format
   sbin/start-dfs.sh
   bin/hdfs dfs -mkdir -p /user/$(whoami)
   #浏览器打开http://ip:50070
   ```

6. 运行案例：grep，worcount

   ```shell
   bin/hdfs dfs -put etc/hadoop input
   bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar grep input output 'dfs[a-z.]+'
   bin/hdfs dfs -cat output/*
   #sbin/stop-dfs.sh
   
   bin/hdfs dfs -put wcinput wcinput
   bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount wcinput wcoutput 
   bin/hdfs dfs -cat wcoutput/*
   ```

7. 配置YARN

   ```shell
   tee etc/hadoop/mapred-site.xml <<-EOF 
   <configuration>
       <property>
           <name>mapreduce.framework.name</name>
           <value>yarn</value>
       </property>
   </configuration>
   EOF
   cp etc/hadoop/yarn-site.xml etc/hadoop/yarn-site.xml.bak
   
   tee etc/hadoop/yarn-site.xml <<-EOF
   <configuration>
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
   </configuration>
   EOF
   sbin/start-yarn.sh
   
   #sbin/stop-yarn.sh
   ```

   

8. 运行案例：grep，wordcount

   ```shell
   hadoop fs -rm -f -r input wcinput output wcoutput
   hadoop fs -put input wcinput
   
   bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount wcinput wcoutput
   hadoop fs -cat wcoutput/*
   
   bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount wcinput wcoutput 'dfs[a-z.]+'
   hadoop fs -cat output/*
    
   
   ```

   

## 集群环境搭建

**先准备三台机器，伪分布环境搭建完成。**

### Steps

1. 修改配置文件

2. 启动

3. 测试

   ```shell
   #http://hadoop101:50070/
   #http://hadoop100:8088/
   #http://hadoop100:19888/
   #http://hadoop102:50090/
   
   ```

   