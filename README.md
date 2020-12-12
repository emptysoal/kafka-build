# Kafka 安装

## 基础环境

- 基于 Ubuntu ，并且已经安装了Java

### 基础环境构建

- 这里用含有 Ubuntu 和 Java 的 docker 镜像环境模拟一台机器，进行 Kafka 的安装

拉取镜像并启动：

```bash
$ docker pull goyalzz/ubuntu-java-8-maven-docker-image
$ docker run -it goyalzz/ubuntu-java-8-maven-docker-image:latest bash
```

在容器中配置ubuntu源：

```bash
root@54d024dccfaa:/# mv /etc/apt/sources.list /etc/apt/sources.list.bak
root@54d024dccfaa:/# echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list
root@54d024dccfaa:/# echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
root@54d024dccfaa:/# echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list
root@54d024dccfaa:/# echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list

# 执行更新和安装命令
root@54d024dccfaa:/# apt-get update
```

安装 vim，telnet：

```bash
root@54d024dccfaa:/# apt-get install vim
root@54d024dccfaa:/# apt-get install telnet
```

把容器生成新镜像，用来安装 Kafka：

```bash
# 退出容器
root@54d024dccfaa:/# exit

$ docker commit 54d024dccfaa myjava:v2
$ docker rm 54d024dccfaa
$ docker images
REPOSITORY                                 TAG
myjava                                     v2
```



## 安装 Zookeeper 和 Kafka

### 安装 Zookeeper 

通过该链接下载：`http://zookeeper.apache.org/releases.html`

本项目下载的为：`apache-zookeeper-3.6.2-bin.tar.gz`

下载后放到本地某目录下，比如：~/kafka

```bash
$ cd ~/kafka
$ docker run -it -v $PWD:/workspace myjava:v2 bash
root@1a3d04a02c77:/# cd workspace
root@1a3d04a02c77:/# tar -zxf apache-zookeeper-3.6.2-bin.tar.gz
root@1a3d04a02c77:/# cp -r apache-zookeeper-3.6.2-bin /usr/local/
root@1a3d04a02c77:/# cd /usr/local/
# 重命名项目目录（个人喜好）
root@1a3d04a02c77:/# mv apache-zookeeper-3.6.2-bin/ zookeeper-3.6.2
root@1a3d04a02c77:/# cd zookeeper-3.6.2

# 修改配置文件
root@1a3d04a02c77:/# cd conf
root@1a3d04a02c77:/# cp zoo_sample.cfg zoo.cfg
root@1a3d04a02c77:/# vim zoo.cfg
```

vim进入后修改如下：

```vim
dataDir=/tmp/zookeeper/data
dataLogDir=/tmp/zookeeper/log
```

创建对应目录：

```bash
root@1a3d04a02c77:/# mkdir /tmp/zookeeper
root@1a3d04a02c77:/# mkdir /tmp/zookeeper/data
root@1a3d04a02c77:/# mkdir /tmp/zookeeper/log
```

配置环境变量：

```bash
root@1a3d04a02c77:/# vim /etc/profile

# 追加下面两行
export ZOOKEEPER_HOME=/usr/local/zookeeper-3.6.2
export PATH=$PATH:$ZOOKEEPER_HOME/bin

root@1a3d04a02c77:/# source /etc/profile
```

把容器生成新镜像，后续专门用来启动 zookeeper：

```bash
# 退出容器
root@1a3d04a02c77:/# exit

$ docker commit 1a3d04a02c77 myzookeeper:v2
$ docker rm 1a3d04a02c77
$ docker images
REPOSITORY                                 TAG
myzookeeper                                v2
```

### 安装 Kafka

通过该链接下载：` http://mirrors.tuna.tsinghua.edu.cn/apache/kafka/0.9.0.0/kafka_2.11-0.9.0.0.tgz`

本项目下载的为：`kafka_2.11-0.9.0.0.tgz`

下载后放到本地某目录下，比如：~/kafka

```bash
$ cd ~/kafka
$ docker run -it -v $PWD:/workspace myjava:v2 bash
root@6299ffc3887b:/# cd workspace
root@6299ffc3887b:/# tar –zxvf  kafka_2.11-0.9.0.0.tgz
root@6299ffc3887b:/# cp -r kafka_2.11-0.9.0.0 /usr/local/
root@6299ffc3887b:/# cd /usr/local/kafka_2.11-0.9.0.0

# 修改配置文件
root@6299ffc3887b:/# vim config/server.properties
```

进入后

```bash
# 在 listeners=PLAINTEXT://:9092 下添加：
advertised.listeners=PLAINTEXT://169.254.207.36:9092

# 大约最下面位置的 zookeeper.connect 改为：
zookeeper.connect=169.254.207.36:2181
```

把容器生成新镜像，后续专门用来启动 kafka：

```bash
# 退出容器
root@6299ffc3887b:/# exit

$ docker commit 6299ffc3887b mykafka:v2
$ docker rm 6299ffc3887b
$ docker images
REPOSITORY                                 TAG
mykafka.                                   v2
```



## 启动

### 启动 zookeeper

- 启动，并进入刚刚创建的 myzookeeper:v2 容器

```bash
$ docker run -it --name myzookeeper -p 2181:2181 myzookeeper:v2 bash
```

- 启动 zookeeper 服务端

```bash
root@9de40f41f0bc:/# cd /usr/local/zookeeper-3.6.2/bin/
root@9de40f41f0bc:/# ./zkServer.sh start
```

***备注*** ：停止服务`./zkServer.sh stop`

启动成功效果如下：

```bash
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.6.2/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

- 启动 zookeeper 客户端

```bash
root@9de40f41f0bc:/# ./zkCli.sh
```

启动成功效果如下：

```bash
Connecting to localhost:2181
...
...
Welcome to ZooKeeper!
...
server localhost/127.0.0.1:2181, session id = 0x1001a1f03790000, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] 
```

- 测试

```bash
$ telnet 169.254.207.36 2181
```

启动成功效果如下：

```bash
Trying 169.254.207.36...
Connected to 169.254.207.36.
Escape character is '^]'.
```

### 启动 Kafka

- 启动，并进入刚刚创建的 mykafka:v2 容器

```bash
$ docker run -it --name mykafka -p 9092:9092 mykafka:v2 bash
```

- 启动 kafka 服务

```bash
root@14c1a82b1f98:/# cd /usr/local/kafka_2.11-0.9.0.0
root@14c1a82b1f98:/# bin/kafka-server-start.sh config/server.properties
```

***备注*** ：停止服务 `bin/kafka-server-stop.sh config/server.properties`

- 测试

```bash
$ telnet 169.254.207.36 9092
```

启动成功效果如下：

```bash
Trying 169.254.207.36...
Connected to 169.254.207.36.
Escape character is '^]'.
```



## 生产消费简单测试

### 生产者

再次启动一个 mykafka 容器

```bash
$ docker run -it mykafka:v2 bash
root@0288fd6eaa5e:/# cd /usr/local/kafka_2.11-0.9.0.0/
```

- 创建 topic (创建了一个名为 test 的 topic)

```bash
root@0288fd6eaa5e:/# bin/kafka-topics.sh --zookeeper 169.254.207.36:2181 --create --topic test --partitions 2 --replication-factor 1
```

- 查看主题列表

```bash
root@0288fd6eaa5e:/# bin/kafka-topics.sh --zookeeper 169.254.207.36:2181 --list
```

- 发布消息

```bash
root@0288fd6eaa5e:/# bin/kafka-console-producer.sh --topic=test --broker-list 169.254.207.36:9092
```

接下来便可向Kafka队列任意发送消息。

### 消费者

- 接收消息

另外启动一个容器，作为消费者

```bash
$ docker run -it mykafka:v2 bash
root@18205532eb54:/# cd /usr/local/kafka_2.11-0.9.0.0/
```

监听队列，接收消息，以0.9版本为界

```bash
root@18205532eb54:/# bin/kafka-console-consumer.sh --bootstrap-server 169.254.207.36:9092 --from-beginning --topic test
```

或：

```bash
root@18205532eb54:/# bin/kafka-console-consumer.sh --zookeeper 169.254.207.36:2181 --from-beginning --topic test
```

**说明** ：一个用的`bootstrap-server`，另一个用的 `zookeeper`

运行之后便能收到生产者发来的消息。

**收发成功，则Kafka环境搭建完成**

