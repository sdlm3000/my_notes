# openTSDB安装记录

[TOC]





### 前期准备

​	设置apt的国内源

```shell
sudo vim /etc/apt/sources.list
# 将其修改为如下内容
deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
```

​	设置root密码

```shell
sudo passwd root
```

​	设置用户sudo免密码

```shell
sudo visudo
# 在末尾添加
sdlm    ALL=(ALL:ALL) NOPASSWD:ALL
```

​	设置host名

```shell
# 分别在3台机器设置主机名（hadoop1/hadoop2/hadoop3）
sudo hostnamectl set-hostname hadoop1  # hadoop2/hadoop3同理

# 编辑hosts文件（3台机器均添加以下内容）
sudo vim /etc/hosts
# 添加：
192.168.120.131 hadoop1
192.168.120.132 hadoop2
192.168.120.133 hadoop3
```

​	设置ssh免密登录

```shell
# 所有主机都安装ssh服务器
sudo apt install openssh-server -y

# 在hadoop1生成密钥（一路回车）
ssh-keygen -t rsa

# 复制公钥到3台机器（包括自身）
ssh-copy-id hadoop1
ssh-copy-id hadoop2
ssh-copy-id hadoop3

```

​	在hadoop1中设置文件分发脚本

```shell
## xsync 脚本代码
## 使用方法：xsync [分发文件或者文件夹]
#!/bin/bash
#1. 判断参数个数
if [ $# -lt 1 ]
then
    echo Not Enough Arguement!
    exit;
fi
#2. 遍历集群所有机器
for host in hadoop1 hadoop2 hadoop3
do
    echo ==================== $host ====================
    #3. 遍历所有目录，挨个发送
    for file in $@
    do
        #4. 判断文件是否存在
        if [ -e $file ]
        then
            #5. 获取父目录
            pdir=$(cd -P $(dirname $file); pwd)
            #6. 获取当前文件的名称
            fname=$(basename $file)
            ssh $host "mkdir -p $pdir"
            rsync -av $pdir/$fname $host:$pdir
        else
            echo $file does not exists!
        fi
    done
done


## 设置为可执行权限
chmod +x xsync

## 复制到/bin下，可以全局使用
sudo cp xscyn /bin

```



​	下载安装包

```shell
# jdk版本
jdk-8u471-linux-x64.tar.gz

# 清华源镜像网址
https://mirrors.ustc.edu.cn/apache/

# 下载如下版本
apache-zookeeper-3.7.2-bin.tar.gz
hadoop-3.3.5.tar.gz
hbase-2.4.18-bin.tar.gz
```



### JDK安装

​	解压JDK，并配置环境变量

```shell
tar -zxvf jdk-8u471-linux-x64.tar.gz -C /opt/module/

# 添加环境变量
vim /etc/profile.d/opentsdb_env.sh
# 添加如下内容
# JAVA HOME
export JAVA_HOME=/opt/module/jdk1.8.0_471
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile

# 验证
java -version
```



### zookeeper安装

​	以hadoop1主机为例，新建/opt/module目录

```shell
cd /opt
mkdir module
# 修改module目录的所在组
sudo chown sdlm:sdlm /opt/module
```

​	将zookeeper解压到/opt/module下

```shell
tar -zxvf apache-zookeeper-3.7.2-bin.tar.gz -C /opt/module
cd /opt/module
mv apache-zookeeper-3.7.2-bin zookeeper-3.7.2
```

​	对zookeeper进行配置

```shell
cd /opt/module/zookeeper-3.7.2
# 新建存储文件夹
mkdir zkData
# 修改zoo.conf
cd ./conf
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
# 主要修改如下内容
dataDir=/opt/module/zookeeper-3.7.2/zkData	# 存储目录
server.1=hadoop1:2888:3888	# zookeeper主机1，其中hadoop1为主机名称
server.2=hadoop2:2888:3888
server.3=hadoop3:2888:3888
	
```

​	将上述zookeeper文件夹复制到hadoop2和hadoop3中

​	配置节点ID

```shell
# 设置id，每个主机设置不同的id号
# hadoop1为1  hadoop2为2  hadoop3为3
# hadoop1
echo "1" > /opt/module/zookeeper-3.7.2/zkData/myid
# hadoop2
echo "2" > /opt/module/zookeeper-3.7.2/zkData/myid
# hadoop3
echo "3" > /opt/module/zookeeper-3.7.2/zkData/myid
```

​	启动zookeeper

```shell
# 在hadoop1、hadoop2、hadoop3中分别启动即可
/opt/module/zookeeper-3.7.2/bin/zkServer.sh start

# 查看状态，它们会根据启动的顺序为 leader 或者 follower
/opt/module/zookeeper-3.7.2/bin/zkServer.sh status

```



### hadoop安装

#### 1. 安装过程

​	在hadoop1中，解压安装包

```shell
tar -zxvf hadoop-3.3.5.tar.gz -C /opt/module/
```

​	配置环境变量

```shell
# 在opentsdb_env.sh中增加hadoop路径
# 添加环境变量
vim /etc/profile.d/opentsdb_env.sh
# 添加如下内容
# HAPOOD_HOME
export HADOOP_HOME=/opt/module/hadoop-3.3.5
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

​	配置 Hadoop 核心文件

```shell
vim /opt/module/hadoop-3.3.5/etc/hadoop/hadoop-env.sh
# 添加如下内容
export JAVA_HOME=/opt/module/jdk1.8.0_471/
export HADOOP_HOME=/opt/module/hadoop-3.3.5/
```

​	hadoop四个配置文件**core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml**的作用：

- core-site.xml：核心全局配置。配置 Hadoop 公共参数，指定 **HDFS 名称节点（NameNode）地址**，设置临时目录、权限、I/O 缓冲等通用配置；
- hdfs-site.xml：HDFS 专用配置。配置 HDFS 自身参数，设置 **文件块副本数**，配置 NameNode、DataNode 数据存放目录，权限检查、访问控制、HDFS 缓存等；
- yarn-site.xml：YARN 资源调度配置。配置 **ResourceManager、NodeManager**，设置容器内存、CPU 限制，配置日志聚集、节点心跳、调度器类型等；
- mapred-site.xml：MapReduce 计算框架配置。配置 MapReduce 运行环境，指定 MR 运行在 **YARN 上**，设置 Map/Reduce 任务内存、CPU、日志等。

#### 2. hadoop HA高可用配置

​	按照如下的方式进行部署配置

|      |   hadoop1   |     hadoop2     |     hadoop3     |
| :--: | :---------: | :-------------: | :-------------: |
|      |  zookeeper  |    zookeeper    |    zookeeper    |
| HDFS |  NameNode   |    NameNode     |                 |
| HFDS |  DataNode   |    DataNode     |    DataNode     |
| YARN |             | ResourceManager | ResourceManager |
| YARN | NodeManager |   NodeManager   |   NodeManager   |
|      |             |                 |                 |

配置core-site.xml，在原来的core-site.xml中的<configuration>内推荐property的配置

```xml

<configuration>
    <!-- 整个集群的名字 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value>
    </property>
    <!-- 指定 hadoop 数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.3.5/data</value>
    </property>
    <!-- 配置 HDFS 网页登录使用的静态用户，不设置会导致Web UI操作无权限  -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>sdlm</value>
    </property>
    <!-- ZooKeeper地址 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>hadoop1:2181,hadoop2:2181,hadoop3:2181</value>
    </property>
</configuration>
```

​	配置hdfs-site.xml

```xml
<configuration>
    <!-- 副本数 -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <!-- 集群名称，和 core-site 保持一致 -->
    <property>
        <name>dfs.nameservices</name>
        <value>mycluster</value>
    </property>
    
    <!-- 两个 NameNode 节点名称 -->
    <property>
        <name>dfs.ha.namenodes.mycluster</name>
        <value>nn1,nn2</value>
    </property>
    <!-- nn1 = hadoop1 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn1</name>
        <value>hadoop1:8020</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.mycluster.nn1</name>
        <value>hadoop1:9870</value>
    </property>
    <!-- nn2 = hadoop2 -->
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn2</name>
        <value>hadoop2:8020</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.mycluster.nn2</name>
        <value>hadoop2:9870</value>
    </property>
    
    <!-- JournalNode 共享日志（自动同步元数据） -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://hadoop1:8485;hadoop2:8485;hadoop3:8485/mycluster</value>
    </property>
    <!-- 设置JournalNode 日志文件路径 -->
    <property>
      	<name>dfs.journalnode.edits.dir</name>
      	<value>/opt/module/hadoop-3.3.5/tmp/journal</value>
    </property>
    <!-- 故障自动切换代理 -->
    <property>
        <name>dfs.client.failover.proxy.provider.mycluster</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    
    <!-- 生产环境HDFS HA隔离配置（推荐） -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <!-- 优先电源隔离，失败则兜底切换 -->
        <value>shell(/bin/true)</value>
    </property>
    <!-- 隔离机制（防止双主） ，使用ssh隔离有点问题，当主设备直接关机了，就会导致ssh失败，导致备用设备启动失败-->
	<!-- 生产环境HDFS HA隔离配置（推荐），默认返回true。一般是远程给主节点远程关机 -->
	<property>
    	<name>dfs.ha.fencing.methods</name>
    	<value>shell(/bin/true)</value>
	</property>

    <!-- 开启自动故障恢复 -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>
</configuration>
```

​	配置yarn-site.xml

```xml
<configuration>
<!-- Site specific YARN configuration properties -->
    <!-- 指定 MR 走 shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- 启用 ResourceManager HA 功能 -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>yarncluster</value>
    </property>
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>hadoop3</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>hadoop2</value>
    </property>
    <!-- 设置访问网址和端口 -->
    <property>
		<name>yarn.resourcemanager.webapp.address.rm1</name>
    	<value>hadoop3:8089</value>
    </property>
    <property>
		<name>yarn.resourcemanager.webapp.address.rm2</name>
    	<value>hadoop2:8089</value>
    </property>
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>hadoop1:2181,hadoop2:2181,hadoop3:2181</value>
    </property>
    <!-- 启用重启 ResourceManager 后保留其恢复状态的功能 -->
    <property>
		<name>yarn.resourcemanager.recovery.enabled</name>
		<value>true</value>
    </property>
</configuration>
```

​	配置mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

​	配置workers，workers里面负责指定谁为dataNode。假设我只需要指定hadoop1和hadoop3为DataNode，则下述文件中删除hadoop2，然后修改hdfs-site.xml中的副本数为2即可。

```shell
vim workers
# 添加如下内容
hadoop1
hadoop2
hadoop3
```

​	 将`/etc/profile.d/opentsdb_env.sh`和`/opt/module/hadoop-3.3.5/`分发给hadoop2和hadoop3。

#### hadoop集群初始化

```shell
# 启动zookeeper，每台机器都执行
/opt/module/zookeeper-3.7.2/bin/zkServer.sh start
# 可以查看一下状态，正常两个follower，一个leader
/opt/module/zookeeper-3.7.2/bin/zkServer.sh status

# 启动JournalNode，每台都执行
hdfs --daemon start journalnode

# 只对 hadoop1 执行格式化ZK
hdfs zkfc -formatZK

# 只对 hadoop1 执行格式化Namenode
hdfs namenode -format

########## 以下操作应该不是必须的
# 启动 hadoop1 的NameNode
hdfs --daemon start namenode
# hadoop2 同步元数据
hdfs namenode -bootstrapStandby
########## 以上操作应该不是必须的

# 启动hadoop1和hadoop2中的 ZKFC
hdfs --daemon start zkfc

# 在hadoop中启动HDFS和YARN
start-dfs.sh
start-yarn.sh
```



#### 运行设置启动脚本

​	在主设备中设置一键启动和停止脚本，以下的`/opt/module/zookeeper-3.7.2`和`/opt/module/hadoop-3.3.5`以具体的路径为准：

```shell
## start-hadoop-ha.sh
#!/bin/bash
echo "======================================="
echo "      Hadoop HA 集群 一键启动脚本       "
echo "         主节点自动控制所有节点          "
echo "======================================="

# ====================== 【必须修改：你的3台机器名】 ======================
nodes="hadoop1 hadoop2 hadoop3"
namenode_nodes="hadoop1 hadoop2"
# =======================================================================

echo -e "\n✅ 1. 启动所有节点 ZooKeeper..."
for node in $nodes; do
  ssh $node "/opt/module/zookeeper-3.7.2/bin/zkServer.sh start"
done

echo -e "\n✅ 2. 一键启动 HDFS（NameNode、DataNode、JournalNode、DFSZKFailoverController）..."
/opt/module/hadoop-3.3.5/sbin/start-dfs.sh

echo -e "\n✅ 3. 一键启动 YARN（ResourceManager、NodeManager）..."
/opt/module/hadoop-3.3.5/sbin/start-yarn.sh

echo -e "\n======================================="
echo "              启动完成！                "
echo "======================================="
jps


## stop-hadoop-ha.sh
#!/bin/bash
echo "停止 Hadoop HA 集群..."

nodes="hadoop1 hadoop2 hadoop3"


echo -e "\n❌ 1. 停止 YARN..."
/opt/module/hadoop-3.3.5/sbin/stop-yarn.sh

echo -e "\n❌ 2. 停止 HDFS..."
/opt/module/hadoop-3.3.5/sbin/stop-dfs.sh

echo -e "\n❌ 3. 停止所有 ZooKeeper..."
for node in $nodes; do
  ssh $node "/opt/module/zookeeper-3.7.2/bin/zkServer.sh stop"
done

echo "集群已全部停止！"

```

​	在启动脚本中远程启动zkServer.sh如果报错找不到JAVA_HOME，则需要在`/opt/module/zookeeper-3.7.2/bin/zkEnv.sh`中的文件首部加入`export JAVA_HOME=/opt/module/jdk1.8.0_471`，每台设备都需要设置。

​	运行成功后查看NameNode和ResourceManager的主备状态。

```shell
# 主设备中使用jps显示如下
3696 DataNode
3314 QuorumPeerMain
3938 JournalNode
4164 DFSZKFailoverController
5956 Jps
3542 NameNode
4616 NodeManager

# 查看所有 NameNode 状态
hdfs haadmin -getAllServiceState

# 查看所有 RM 状态
# 实际可能会出现yarn与系统中的npm中的yarn指令冲突，需要加上yarn的路径，如
# /opt/module/hadoop-3.3.5/bin/yarn rmadmin -getAllServiceState
yarn rmadmin -getAllServiceState

```

​	访问 Web 界面：`http://hadoop1:9870`（HDFS）、`http://hadoop2:8089`（YARN），能够正常访问代表hadoop部署成功

#### 高可用验证

##### c++程序示例

​	配置环境变量

```shell
# 添加环境变量
vim /etc/profile.d/opentsdb_env.sh

# C++_HOME
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$JAVA_HOME/jre/lib/amd64/server:$LD_LIBRARY_PATH
# 仅交互式终端才执行 Hadoop classpath，桌面不加载
if [ -n "$PS1" ] && [ -d $HADOOP_HOME ]; then
    export CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath --glob):$CLASSPATH
fi
```

​	

```c++
## c++程序，100ms写入一个文件，写100个
#include <iostream>
#include <cstring>
#include <string>
#include <chrono>
#include <thread>
#include "hdfs.h"

using namespace std;

int main() {
    // ==============================================
    // 🔥 HA 高可用写法：连接集群名，不连具体机器
    // ==============================================
    const char* hdfsHost = "mycluster";  // 你 hdfs-site.xml 里的 nameservice
    int hdfsPort = 0;                    // HA 模式必须写 0

    // 连接 HDFS HA 集群
    hdfsFS fs = hdfsConnect(hdfsHost, hdfsPort);
    if (!fs) {
        std::cerr << "❌ 连接 HDFS HA 失败！" << std::endl;
        return -1;
    }
    std::cout << "✅ 连接 HDFS HA 成功（自动识别主节点）" << endl;

    // 文件名从 1 开始递增
    int fileIndex = 1;

    // 无限循环写入（想停止按 Ctrl+C）
    while (fileIndex <= 100) {
        // ===================== 拼接递增文件名 =====================
        string fileName = "/user/test/ha_test" + to_string(fileIndex) + ".txt";
        const char* path = fileName.c_str();

        // 写入内容（带序号）
        string data = "这是第 " + to_string(fileIndex) + " 个文件，100ms 自动生成！";

        // ===================== 写入文件 =====================
        hdfsFile writeFile = hdfsOpenFile(fs, path, O_WRONLY | O_CREAT, 0, 0, 0);
        if (!writeFile) {
            std::cerr << "❌ 打开文件失败：" << path << endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            continue;
        }

        // 写入数据
        hdfsWrite(fs, writeFile, data.c_str(), data.size());
        cout << "✅ 写入成功：" << path << endl;
        hdfsCloseFile(fs, writeFile);

        // 序号 +1
        fileIndex++;

        // ===================== 等待 100ms =====================
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }

    // 关闭连接（程序正常退出才会走到）
    hdfsDisconnect(fs);
    return 0;
}

## 编译
g++ hdfs_ha.cpp -o hdfs_ha -I$HADOOP_HOME/include -L$HADOOP_HOME/lib/native -L$JAVA_HOME/jre/lib/amd64/server -lhdfs -ljvm

## 运行
./hdfs_ha > text.txt &
```

##### 高可用验证流程

​	常用指令：

```shell
# 查看hadoop目录内容，其他常见操作与基本shell指令类似，将-ls替换为其他即可
hdfs dfs -ls [hadoop目录]

# 上传整个文件或文件夹
hdfs dfs -put [本地文件/本地文件夹] [hadoop目录]
# 下载
hdfs dfs -get [HDFS路径] [本地路径]

```



​	循环存储验证：

- 正常启动hadoop集群，确认主namenode节点为hadoop1
- 运行上述c++程序后，快速kill掉hadoop1中namenode
- 使用`hdfs dfs -ls /user/test/`查看是否有100个文件，正常在kill了hadoop1中namenode后，程序还是能正常存放文件的



​	存储文件副本自动备份验证：

- 在hadoop1挂掉后，在hdfs中上传一个大文件
- 然后重启集群，查看在hadoop1中是否将大文件自动备份

```shell
# 每个 HDFS 文件被切成 128MB（默认）块
# 块文件在 DataNode 上叫：blk_xxxx，带校验文件
# 如下所示的路径中有很多subdir，里面有很多blk
/opt/module/hadoop-3.3.5/data/dfs/data/current/BP-1737057284-192.168.120.131-1775460042390/current/finalized

# 在集群重启前查看hadoop1上述路径的文件夹大小
du -sh /opt/module/hadoop-3.3.5/data/dfs/data/current/BP-1737057284-192.168.120.131-1775460042390/current/finalized
# 重启后再查看，一般会自动将大文件备份，能正常肯定该文件夹变大

```



### Hbase安装

​	在hadoop1中，解压安装包

```shell
tar -zxvf hbase-2.4.18-bin.tar.gz -C /opt/module/
cd /opt/module
mv hbase-2.4.18 hbase
```

​	配置环境变量

```shell
sudo vim /etc/profile.d/opentsdb_env.sh
# 添加如下内容
# HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin


    <property>
        <name>hbase.unsafe.stream.capability.enforce</name>
        <value>false</value>
    </property>

    <property>
        <name>hbase.wal.provider</name>
        <value>filesystem</value>
    </property>

    <property>
        <name>hbase.regionserver.wal.async.sink.enabled</name>
        <value>false</value>
    </property>

```





#### hbase的集群启动脚本



```shell
## start-hadoop-ha.sh
#!/bin/bash
echo "======================================="
echo "      Hadoop HA 集群 一键启动脚本       "
echo "         主节点自动控制所有节点          "
echo "======================================="

# ====================== 【必须修改：你的3台机器名】 ======================
nodes="hadoop1 hadoop2 hadoop3"
namenode_nodes="hadoop1 hadoop2"
# =======================================================================

echo -e "\n✅ 1. 启动所有节点 ZooKeeper..."
for node in $nodes; do
  ssh $node "/opt/module/zookeeper-3.7.2/bin/zkServer.sh start"
done

echo -e "\n✅ 2. 一键启动 HDFS（NameNode、DataNode、JournalNode、DFSZKFailoverController）..."
/opt/module/hadoop-3.3.5/sbin/start-dfs.sh

echo -e "\n✅ 3. 一键启动 YARN（ResourceManager、NodeManager）..."
/opt/module/hadoop-3.3.5/sbin/start-yarn.sh

echo -e "\n✅ 4. 启动 HBase 集群（HMaster + HRegionServer）..."
/opt/module/hbase/bin/start-hbase.sh

echo -e "\n======================================="
echo "              启动完成！                "
echo "======================================="
jps


## stop-hadoop-ha.sh
#!/bin/bash
echo "停止 Hadoop HA 集群..."

nodes="hadoop1 hadoop2 hadoop3"

echo -e "\n❌ 1. 停止 HBase 集群..."
/opt/module/hbase/bin/stop-hbase.sh

echo -e "\n❌ 2. 停止 YARN..."
/opt/module/hadoop-3.3.5/sbin/stop-yarn.sh

echo -e "\n❌ 3. 停止 HDFS..."
/opt/module/hadoop-3.3.5/sbin/stop-dfs.sh

echo -e "\n❌ 4. 停止所有 ZooKeeper..."
for node in $nodes; do
  ssh $node "/opt/module/zookeeper-3.7.2/bin/zkServer.sh stop"
done

echo "集群已全部停止！"
```

