# Hadoop的安装和基本配置
### 配置java环境
下载Jdk（1.6以上）
### 配置Hadoop
下载hadoop 2.x（最新的binary包）

配置环境变量
```sh
export HADOOP_INSTALL=/opt/hadoop-2.7.2
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib/native"
#HADOOP VARIABLES END

export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/
export JRE_HOME=${JAVA_HOME}/jre

export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export PATH=${PATH}:${JAVA_HOVE}/bin:$JRE_HOME/bin:$MAVEN_HOME:$HADOOP_INSTALL/bin
```
### 配置ssh免登陆
集群上每一台机器都应当配置SSH单次登陆

ubuntu自带的ssh不满足需要，需要安装ssh：
```sh
$ sudo apt-get install ssh
```

免密码ssh设置
```sh
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```

至此，hadoop独立模式的环境已经基本配置完成。

# Hadoop的三种模式
## 独立模式配置
hadoop默认的模式，工作在一台主机上，仅用于mapReduce的调试
## 伪分布模式配置
在单主机上搭建的分布模式，主要用于测试，对位于hadoop包下的etc/下的相关xml文件需进行以下配置：
```xml
 <?xml version="1.0"?>
    <!-- core-site.xml -->
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://localhost/</value> 
        </property>
    </configuration>
<?xml version="1.0"?>
    <!-- hdfs-site.xml -->
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>1</value> 
        </property>
    </configuration>
<?xml version="1.0"?>
    <!-- mapred-site.xml -->
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value> 
        </property>
    </configuration>
<?xml version="1.0"?>
    <!-- yarn-site.xml -->
    <configuration>
        <property>
            <name>yarn.resourcemanager.hostname</name>
            <value>localhost</value> 
        </property>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value> 
        </property>
    </configuration>
```
其中，core-site.xml配置的是namenode的名称已经内网ip

## 集群模式的配置
以配置阿里云服务器为例

#### 修改主机名和登陆提示

修改主机名和登陆提示是为了后期在部署集群的时候能够统一命名规则

修改ubuntu系统登陆提示
```sh
$ sudo vim /etc/passwd
```
找到当前用户信息，修改为统一命名规则的登陆名称,登陆提示需要重启系统才会变更.

修改ubuntu主机名称
```sh
$ sudo vim /etc/hostname
```
直接修改为所需的主机名称，例如改为s0，
直接通过hostname查看修改结果
#### 使用符号链接实现配置分离
通过配置软链接，就无需每次都配置--config或者设置环境变量来指定启动的配置了
```sh
$ ln -s hadoop_clutser hadoop
```

### 配置集群内每台主机的Hosts文件
集群内每台主机的hosts，由于其格式都相同，即
```sh
主机ip地址 主机名
```
通过远程拷贝实现同步，将一台主机上改好的hosts文件通过scp指令拷贝到集群内其他主机上，命令为：

```sh
$ sudo scp /etc/hosts root@目标主机的ip:/etc/
```


# HDFS分析
特点：
* 写入成本高，应用于一次写入，多次读取的情况，
* namenode至关重要，且只有一个，可以考虑用RAID
* datanode不需要RAID（hadoop有副本机制代替，默认有两个副本）
* 切割尺寸：128M（blocksize）
* 定点节点位置感知
### namenode和datanode


# MapReduce
MapReduce: map + reduce， 大数据量（一般为纯文本） 

每一次map task  + reduce task 的过程就是一次作业（job）
* job: 作业
* task： 任务
### Maper

K-V --> map --> K-V --> shuffle -->
### Reducer


