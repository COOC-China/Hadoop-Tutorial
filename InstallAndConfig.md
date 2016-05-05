# Hadoop的安装和基本配置
## 配置java环境
下载Jdk（1.6以上）
## 安装Hadoop
下载hadoop 2.7.2

将包解压至其他目录/opt/ (同下面环境变量一致)
```sh
$ tar zxf hadoop-2.7.2.tar.gz
```
配置环境变量

将以下变量添加至 /etc/profile 或者 ~/.bashrc 中

（安装目录为/opt/hadoop-2.7.2)
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
保存后使之生效
```sh
$ source filname
```

## 配置ssh免登陆

###配置ssh密钥

通过添加公私密钥实现免密码登陆

生成密钥
```sh
$ ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys
```

至此，hadoop独立模式的环境已经基本配置完成。

配置后尝试登陆

```sh
$ ssh localhost
```

# Hadoop的三种模式
## 独立模式配置
hadoop默认的模式，工作在一台主机上，仅用于mapReduce的调试，默认解压后的状态，不需要配置文件
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

## 集群模式的配置
以配置阿里云服务器为例

###修改xml配置文件
```xml
 <?xml version="1.0"?>
    <!-- core-site.xml -->
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://hostname/</value> 
        </property>
    </configuration>
    
<?xml version="1.0"?>
    <!-- hdfs-site.xml -->
    <configuration>
        <property>
            <name>dfs.replication</name>
            <value>3</value> 
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
其中，core-site.xml配置的是namenode的名称以及内网ip

### 修改主机名和登陆提示

修改主机名和登陆提示是为了后期在部署集群的时候能够统一命名规则

####修改ubuntu系统登陆提示
```sh
$ sudo vim /etc/passwd
```
找到当前用户信息，修改为统一命名规则的登陆名称,登陆提示需要重启系统才会变更.

####修改ubuntu主机名称
```sh
$ sudo vim /etc/hostname
```
直接修改为所需的主机名称，例如改为hadoop1，
直接通过hostname命令查看修改结果


### 配置集群内每台主机的Hosts文件
集群内每台主机的hosts，由于其格式都相同，即
```sh
主机ip地址 主机名
例如
10.252.6.1 hadoop1

10.252.6.10 hadoop2

10.252.5.232 hadoop3

10.252.5.244 hadoop4

10.252.6.0 hadoop5

10.252.6.28 hadoop6

```
通过远程拷贝实现同步，将一台主机上改好的文件通过scp指令拷贝到集群内其他主机上，命令为：

```sh
1、获取远程服务器上的文件

    $ scp linksprite@hadoop1:/home/linksprite/lnmp0.4.tar.gz ./lnmp0.4.tar.gz

2、获取远程服务器上的目录

    $ scp -r linksprite@hadoop1:/linksprite/lnmp0.4/ ./lnmp0.4/


3、将本地文件上传到服务器上

    $ scp ./lnmp0.4.tar.gz linksprite@hadoop1:/home/linksprite/lnmp0.4.tar.gz

4、将本地目录上传到服务器上

    $ scp -r ./lnmp0.4/ linksprite@hadoop1:/home/linksprite/lnmp0.4/

参数解释
    -P 2222 指向更改后的ssh端口
    -r 参数表示递归复制
    linksprite@hadoop1 表示使用linksprite用户登录远程服务器hadoop1，

```
需要拷贝的文件有环境变量文件，/etc/hosts文件, $HADOOP_INSTALL/etc/hadoop/目录下的配置文件,

以及~/.ssh目录下的公私密钥（公钥放入目录后通过cat id_rsa.pub >> authorized_keys命令加入密钥，通过

ssh hostname登陆来验证),以上拷贝至各个主机的对应目录。

#### 使用符号链接实现配置分离
通过配置软链接，就无需每次都配置--config或者设置环境变量来指定启动的配置了
```sh
$ cd $HADOOP_INSTALL/etc/
$ ln -s hadoop_clutser hadoop
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


