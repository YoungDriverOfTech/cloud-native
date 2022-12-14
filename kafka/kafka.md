# Login Server  

```shell
ssh -i ./pem ec2-user@publicAddress
```

# 安装
## 环境搭建
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
tar -zxvf apache-zookeeper-3.5.7.tar.gz -C ../install/
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
