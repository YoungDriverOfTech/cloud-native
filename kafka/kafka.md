# Login Server  

```shell
ssh -i ./pem ec2-user@publicAddress
```

# 安装
## 环境搭建

### 配置java环境
```shell
# 切换成root用户
sudo su -
# 设置新密码
passwd


# 安装一个复制的前置软件
yum install -y lrzsz

# 复制jdk zookeeper kafka安装包到服务器 /opt/sofrware
# 因为ec2-user是登录时候必须用的，所以不能直接上传到/opt/目录，得先传到ec2-user家目录，在拷贝过去才行
scp -i ./pem apache-zookeeper-3.5.7.tar.gz jdk-8u202-linux-x64.tar.gz kafka_2.11-2.4.0.tgz ec2-user@ip:/home/ec2-user/software

# 在/opt下创建install目录，把所有的安装包解压在这里面, 剩下两个同理
tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C ../install/
tar -zxvf jdk-8u202-linux-x64.tar.gz -C ../install/
tar -zxf kafka_2.11-2.4.0.tgz -C ../install/

# 配置jdk环境变量
# 进入jdk目录，复制其安装路径  /opt/install/jdk1.8.0_202
# 进入/etc/profile, 最尾部加上

## JAVA_HOME
export JAVA_HOME=/opt/install/jdk1.8.0_202

## PATH
export PATH=$PATH:$JAVA_HOME/bin

# 手动刷新配置后，确认java -version  (单横线)
source /etc/profile

```


### 配置并且启动zookeeper
```shell
# 进入zookeeper文件夹，修改配置文件
cd /opt/install/apache-zookeeper-3.5.7/conf

# 默认的配置文件是zoo.cfg, 有一个样例 zoo_sample.cfg，复制一份
# dataDir 可以修改，一般放在大空间目录下
dataDir=/tmp/zookeeper

# 进入bin目录查看启动命令。启动失败的话，需要重新下载zookpper的包，名字里面要带有bin的才能正常启动
./zkServer.sh start


```

### 配置并且启动kafka
- 概念普及  
> Topic: 虚拟概念，由1到多个partitions组成  
> partition: 实际消息存储单位  
> Producer: 消息生产者  
> Consumer: 消息消费者  

- 配置以及启动
```shell
# 要配置ec2 安全组，把9092端口放开

# 进入config文件夹，修改server.properties
# listeners=PLAINTEXT://:9092 修改为本机器 Public IPv4 DNS
listeners=PLAINTEXT://Public IPv4 DNS:9092

# #advertised.listeners=PLAINTEXT://your.host.name:9092 修改为本机 Public IPv4 DNS
advertised.listeners=PLAINTEXT://Public IPv4 DNS:9092

# kafka存放数据到zookeeper的路径
log.dirs=/tmp/kafka-logs

# 因为zoo装载本机，所以地址用localhost也行
zookeeper.connect=localhost:2181
```

## kafka 基本命令  
```shell
# 1. 启动kafka (启动后可使用 ps rf | grep kafka  确认)  加上&是让他后台启动
./bin/kafka-server-start.sh config/server.properties &

# 2. 停止kafka
./bin/kafka-server-stop.sh

# 3. 创建topic
./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic first-topic

# 4. 查看topic
./bin/kafka-topics.sh --list --zookeeper localhost:2181

# 5. 发送消息 (因为启动的时候用了dns，所以发送消息也要换成dns)
./bin/kafka-console-producer.sh --broker-list Public IPv4 DNS:9092 --topic first-topic

# 6. 消费消息
./bin/kafka-console-consumer.sh --bootstrap-server Public IPv4 DNS:9092 --topic first-topic --from-beginning
```

# kakfka的API  
## AdminClient API  

### AdminClient  
AdminClient客户端对象

```java
package com.liang.kafkapricatice.admin;

import org.apache.kafka.clients.admin.AdminClient;
import org.apache.kafka.clients.admin.AdminClientConfig;

import java.util.Properties;

public class AdminSimple {

    public static void main(String[] args) {
        AdminClient adminClient = AdminSimple.adminClient();
        System.out.println("adminClient = " + adminClient);
    }

    /*
    * 设置admin client
    * */
    public static AdminClient adminClient() {
        // kafka的配置
        Properties properties = new Properties();

        // 集群地址
        properties.setProperty(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka Address:9092");

        return AdminClient.create(properties);
    }
}
```


### NewTopic
创建Topic
```java
public static void createTopic() {
        AdminClient adminClient = AdminSimple.adminClient();

        // 副本因子
        short rs = 1;

        // 第二个参数是分区数量，第三个是副本数量
        NewTopic newTopic = new NewTopic(TOPIC_NAME, 1, rs);

        // createTopics接受数组
        CreateTopicsResult topics = adminClient.createTopics(List.of(newTopic));
        System.out.println("topics = " + topics);
    }
```

### CreateTopicsResult
创建Topic的返回结果

### ListTopicsResult & ListTopicsOptions 
查询Topic列表  
查询Topic列表及选项 
```java
    /*
    * kafka查找
    * */
    public static void topicLists() throws Exception {
        AdminClient adminClient = adminClient();

        // 把内部topic一起打印出来
        ListTopicsOptions options = new ListTopicsOptions();
        options.listInternal(true);

        // 不加options，不打印内部topic
//        ListTopicsResult listTopicsResult = adminClient.listTopics();

        // 加上options，把内部topic也能打出来
        ListTopicsResult listTopicsResult = adminClient.listTopics(options);

        Set<String> names = listTopicsResult.names().get();
        names.forEach(System.out::println);
    }
```


### DescribeTopicsResult
查询Topics

### DescribeConfigsResult
查询Topics配置项