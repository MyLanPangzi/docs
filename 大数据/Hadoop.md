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

**先准备三台机器，伪分布环境搭建完成，三个节点之间hosts文件记得修改，网络互通。**

| hadoop100           | hadoop101    | hadoop102             |
| ------------------- | ------------ | --------------------- |
| **resourcemanager** | **namenode** | **secondarynamenode** |
| nodemanager         | nodemanager  | nodemanager           |
| datanode            | datanode     | datanode              |
| historyserver       |              |                       |

### Steps

1. 格式化节点

   ```shell
   #删除logs，data目录
   ssh hadoop100 rm -rf /opt/module/hadoop-2.10.0/data /opt/module/hadoop-2.10.0/logs
   ssh hadoop102 rm -rf /opt/module/hadoop-2.10.0/data /opt/module/hadoop-2.10.0/logs
   ssh hadoop101 rm -rf /opt/module/hadoop-2.10.0/data /opt/module/hadoop-2.10.0/logs && hdfs namenode -format
   ```
   
2. 配置时间同步

   ```shell
   #选择一台节点作为主服务器，hadoop100
   vim /etc/chrony.conf
   #打开此行
   #allow 192.168.0.0/16
   systemctl restart chronyd
   #测试
   date -s "20:00:00"
   #等一分钟后执行
   chronyc makestep 1 -1
   crontab -e 
   */1 * * * * chronyc makestep 1 -1
   
   #修改其他节点配置
   vim /etc/chrony.conf
   #注释pool那行，增加
   server hadoop100 iburst
   
   crontab -e 
   */1 * * * * chronyc makestep 1 -1
   ```

3. 准备分发脚本

   ```shell
   #分发之前先ssh互通各台机器，注意用户
   ssh-keygen -t rsa
   #ssh-copy-id root@haddop100
   ssh-copy-id haddop100
   ssh-copy-id haddop101
   ssh-copy-id haddop102
   
   tee ~/bin/xsync <<-EOF
   #!/bin/bash
   #1 获取输入参数个数，如果没有参数，直接退出
   pcount=$#
   if [ $pcount -eq 0 ]; then
   echo no args;
   exit;
   fi
   
   #2 获取文件名称
   p1=$1
   fname=`basename $p1`
   echo fname=$fname
   
   #3 获取上级目录到绝对路径
   pdir=`cd -P $(dirname $p1) || exit; pwd`
   echo pdir=$pdir
   
   #4 获取当前用户名称
   user=`whoami`
   
   #5 循环
   for((host=100; host<103; host++)); do
           echo ------------------- hadoop$host --------------
           rsync -av $pdir/$fname $user@hadoop$host:$pdir
   done
   EOF
   
   ```

   

4. 修改配置文件

   ```shell
   #DFS配置
   tee $HADOOP_HOME/etc/hadoop/core-site.xml <<-EOF
   <configuration>
   <!-- 指定HDFS中NameNode的地址 -->
       <property>
           <name>fs.defaultFS</name>
           <value>hdfs://hadoop101:9000</value>
       </property>
   
   <!-- 指定Hadoop运行时产生文件的存储目录 -->
       <property>
           <name>hadoop.tmp.dir</name>
           <value>/opt/module/hadoop-2.10.0/data/tmp</value>
       </property>
   </configuration>
   EOF
   
   #HDFS配置
   tee $HADOOP_HOME/etc/hadoop/hdfs-site.xml <<-EOF
   <configuration>
   	<!-- namenode 工作目录 -->
       <property>
           <name>dfs.namenode.name.dir</name> 					
           <value>
             file:///${hadoop.tmp.dir}/dfs/name1,file:///${hadoop.tmp.dir}/dfs/name2
           </value>
       </property>
       <!-- 指定HDFS副本的数量 -->
       <property>
           <name>dfs.replication</name>
           <value>3</value>
       </property>
       <!-- 指定Hadoop辅助名称节点主机配置 -->
       <property>
           <name>dfs.namenode.secondary.http-address</name>
           <value>hadoop102:50090</value>
       </property>
   </configuration>
   EOF
   
   #YARN配置
   tee $HADOOP_HOME/etc/hadoop/yarn-site.xml <<-EOF
   <configuration>
       <property>
           <name>yarn.nodemanager.aux-services</name>
           <value>mapreduce_shuffle</value>
       </property>
   
   <!-- 指定YARN的ResourceManager的地址 -->
       <property>
           <name>yarn.resourcemanager.hostname</name>
           <value>hadoop100</value>
       </property>
   </configuration>
   EOF
   
   #mapreduce配置
   tee $HADOOP_HOME/etc/hadoop/mapred-site.xml <<-EOF
   <configuration>
           <!-- 历史服务器端地址 -->
       <property>
           <name>mapreduce.jobhistory.address</name>
           <value>hadoop100:10020</value>
       </property>
       <!-- 历史服务器web端地址 -->
       <property>
           <name>mapreduce.jobhistory.webapp.address</name>
           <value>hadoop100:19888</value>
       </property>             
       <property>
           <name>mapreduce.framework.name</name>
           <value>yarn</value>
       </property>
   </configuration>
   EOF
   
   #slaves配置
   tee $HADOOP_HOME/etc/hadoop/slaves <<-EOF
   hadoop100
   hadoop101
   hadoop102
   EOF
   
   #分发配置
   ~/bin/xsync.sh  $HADOOP_HOME/etc/hadoop/
   ```

5. 准备启动脚本，检测脚本

   ```shell
   tee ~/bin/start.sh <<-EOF
   #!/bin/bash
   ssh hadoop100 /opt/module/hadoop-2.10.0/sbin/mr-jobhistory-daemon.sh start historyserver
   ssh hadoop100 /opt/module/hadoop-2.10.0/sbin/start-yarn.sh
   ssh hadoop101 /opt/module/hadoop-2.10.0/sbin/start-dfs.sh
   ./myjps
   EOF
   
   tee ~/bin/myjps.sh <<-EOF
   #!/bin/bash
   for i in hadoop100 hadoop101 hadoop102
   do
     echo "--------------------$i--------------"
     ssh $i /opt/module/jdk-13.0.2/bin/jps
     echo "--------------------$i--------------"
     echo -e "\n"
   done;
   EOF
   #启动
   ~/bin/start.sh
   
   ```

   

6. 测试

   ```shell
   #http://hadoop101:50070/
   #http://hadoop100:8088/   
   #http://hadoop100:19888/
   #http://hadoop102:50090/
   tee input <<-EOF
   hello world
   EOF
   
   hdfs dfs -mkdir -p /user/$(whoami)/
   hdfs dfs -put input
   hadoop jar /share/hadoop/mapreduce/hadoop-mapreduce-examples.jar wordcount input output 
   hdfs dfs -cat output/*
   ```

## 新增数据节点

### Steps

1. 准备好服务器
2. 搭建好hadoop单机环境
3. 拷贝namenode下的hadoop/etc目录下的文件
4. 删除data，logs目录
5. 启动datanode

## 退役数据节点

### Steps

1. 白名单退役（不推荐）

   ```shell
   #修改hdfs-site.xml，新增下面的配置
   <property>
   <name>dfs.hosts</name>
   <value>/opt/module/hadoop-2.10.0/etc/hadoop/dfs.hosts</value>
   </property>
   
   #新增etc/hadoop/dfs.hosts文件
   tee etc/hadoop/dfs.hosts <<-EOF
   hadoop100
   hadoop101
   hadoop102
   EOF
   
   #分发
   xsync etc/hadoop && hdfs dfsadmin -refreshNodes
   
   ```

2. 黑名单退役

   ```shell
   #修改hdfs-site.xml
   <property>
   	<name>dfs.hosts.exclude</name>
         <value>/opt/module/hadoop-2.10.0/etc/hadoop/dfs.hosts.exclude</value>
   </property>
   
   tee etc/hadoop/dfs.hosts.exclude <<-EOF
   hadoop103
   EOF
   
   xsync etc/hadoop 
   hdfs dfsadmin -refreshNodes
   
   ```
```
   

## DataNode多目录配置

​```shell
#hdfs-site.xml
<property>
        <name>dfs.datanode.data.dir</name>
<value>file:///${hadoop.tmp.dir}/dfs/data1,file:///${hadoop.tmp.dir}/dfs/data2</value>
</property>


```

| hadoop100 | hadoop101 | hadoop102 | hadoop103 | hadoop104 | hadoop105 | hadoop106 |
| --------- | --------- | --------- | --------- | --------- | --------- | --------- |
| RM        | RM        | NN        | NN        | ZK        | ZK        | ZK        |
| History   |           | ZKFC      | ZKFC      | DN        | DN        | DN        |
|           |           |           |           | NM        | NM        | NM        |
|           |           |           |           | JN        | JN        | JN        |
|           |           |           |           |           |           |           |
|           |           |           |           |           |           |           |
|           |           |           |           |           |           |           |

