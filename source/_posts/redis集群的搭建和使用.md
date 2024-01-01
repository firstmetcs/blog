---
title: redis集群的搭建和使用
date: 2024-01-01 18:30:20
tags:
- 原创
- Redis
- Redis sentinel
categories:
- Redis
---
# 前言
本篇知识分享主要讲述使用docker搭建redis集群，其中包含redis简单主从模式、哨兵模式和springboot整合redis集群等，适合初学者入门搭建和使用参考。
## 基础服务环境
本篇教程基于docker容器化搭建redis集群，需要使用docker环境。
基础环境细节如下：
* Docker环境
* Redis环境（docker latest）
* Java环境
* Maven环境
### 搭建docker环境
* 卸载旧版本
```bash
yum remove docker docker-common docker-selinux
```
* 安装需要的依赖包
```bash
yum install -y yum-utils device-mapper-persistent-data
```
* 配置稳定仓库
```bash
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
* 安装
```bash
yum install docker-ce
```
* 启动docker
```bash
systemctl start docker
```
* 验证安装是否成功
```bash
docker -v
```
### docker环境配置
为避免docker容器运行时使用基于docker的局域网环境信息导致后续的springboot使用哨兵模式调用失败，启动时需要添加net网络模式选项：
```
–net=host
```
这样容器就和宿主机共用网络

# Redis简单主从模式
## 搭建基础redis环境
### 获取redis镜像
下面的命令会拉取最新的官方版本的redis镜像
```bash
docker pull redis
```
查看镜像
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041197194105.jpg)

### 获取并修改redis配置文件
下面的命令会拉取最新的官方版本的redis镜像
redis官方提供了一个配置文件样例，通过wget工具下载下来。我用的root用户，就直接下载到用户主目录里了。
```bash
wget http://download.redis.io/redis-stable/redis.conf
```
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041197616545.jpg)

配置文件修改以下几项：
```conf
# 注释这一行，表示Redis可以接受任意ip的连接
# bind 127.0.0.1 

# 关闭保护模式
protected-mode no 

# 设定密码(可选，如果这里开启了密码要求，slave的配置里就要加这个密码. 只是练习配置，就不使用密码认证了)
# requirepass masterpassword 

# 配置日志路径，为了便于排查问题，指定redis的日志文件目录
logfile "/var/log/redis/redis.log"
```
### 使用配置文件启动redis环境
启动时复制配置文件到容器中并使用配置文件启动redis server
```bash
docker run -d -p 6379:6379 --name redis -v /hzero/repo/redis/redis-master.conf:/usr/local/redis.conf redis redis-server /usr/local/redis.conf
```
其中/hzero/repo/redis/redis-master.conf为下载并修改的配置文件的位置，/usr/local/redis.conf是docker容器中配置文件的位置，redis-server /usr/local/redis.conf表示在容器启动时使用配置文件启动redis server
### 注意的坑
有时候会发现docker使用配置文件启动redis启动失败了，其实并不是启动失败了，而是配置文件中添加了以下配置：
```conf
# 让redis服务后台运行
daemonize yes
```
daemonize yes，他的作用是是否开启守护进程模式，在该模式下，redis会在后台运行，并将进程pid号写入至redis.conf选项pidfile设置的文件中，此时redis将一直运行，除非手动kill该进程
那么通过配置文件的方式启动redis肯定就无法开启守护进程模式，所以导致配置文件中daemonize yes的情况下，无法通过配置文件启动
## 搭建主从模式redis集群
### 修改主节点master配置
配置文件修改以下几项：
```conf
# 注释这一行，表示Redis可以接受任意ip的连接
# bind 127.0.0.1 

# 关闭保护模式
protected-mode no 

# 设定密码(可选，如果这里开启了密码要求，slave的配置里就要加这个密码. 只是练习配置，就不使用密码认证了)
# requirepass masterpassword 

# 配置日志路径，为了便于排查问题，指定redis的日志文件目录
logfile "/var/log/redis/redis.log"
```
### 修改从节点slave配置
配置文件修改以下几项：
```conf
# 注释这一行，表示Redis可以接受任意ip的连接
# bind 127.0.0.1 

# 关闭保护模式
protected-mode no 

# 设定密码(可选，如果这里开启了密码要求，slave的配置里就要加这个密码)
requirepass masterpassword 

# 设定主库的密码，用于认证，如果主库开启了requirepass选项这里就必须填相应的密码
masterauth <master-password>

# 设定master的IP和端口号，redis配置文件中的默认端口号是6379
# 低版本的redis这里会是slaveof，意思是一样的，因为slave是比较敏感的词汇，所以在redis后面的版本中不在使用slave的概念，取而代之的是replica
# 将35.236.172.131做为主，其余两台机器做从。ip和端口号按照机器和配置做相应修改。
replicaof 35.236.172.131 6379

# 配置日志路径，为了便于排查问题，指定redis的日志文件目录
logfile "/var/log/redis/redis.log"
```
### 启动容器
分别在主机和从机上按照上面的方法建立好配置文件，检查无误后就可以开始启动容器了。
我们在三台机器上分别将容器别名指定为redis, redis-slave，这样便于区分与说明，docker通过--name参数来指定容器的别名。redis是master上容器的别名，redis-slave是slave上的别名。
只是要注意master的配置文件和slave不同。不过首先要启动主服务器，也就是redis容器。然后再启动redis-slave。
在此处为了区分主从两个容器，分别给主服务器6379端口和从服务器6380端口
启动主服务器：
```bash
docker run -d -p 6379:6379 --name redis -v /hzero/repo/redis/redis-master.conf:/usr/local/redis.conf redis redis-server /usr/local/redis.conf
```
然后启动从服务器
```bash
docker run -d -p 6380:6379 --name redis-slave -v /hzero/repo/redis/redis-slave.conf:/usr/local/redis.conf redis redis-server /usr/local/redis.conf
```
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041191763913.jpg)

### 测试（验证主从复制）
使用以下命令进入redis的容器：
```bash
docker exec -it redis bash
```
使用redis-cli进入容器，查看现有的key，可以看到为空
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041135641014.jpg)

在此服务器环境设置一个新key并查看：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041135822950.jpg)

此时只在主服务器上设置了key，在从服务器应该自动复制过去一份，我们使用以下命令进入从服务器：
```bash
docker exec -it redis-slave bash
```
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041136048755.jpg)
可以看到从服务器已经复制了主服务器上的key
至此，主从集群模式的redis已经搭建成功。
# Redis哨兵模式的集群
## 简介
### 前言
之前介绍了用docker来搭建redis主从环境，但这只是对数据添加了从库备份(主从复制)，当主库down掉的时候，从库是不会自动升级为主库的，也就是说，该redis主从集群并非是高可用的。
但是如果主发生故障，需要人工手动切换从机为主机。这种切换工作不仅浪费人力资源，更大的影响是主从切换期间这段时间redis是无法对外提供服务的。因此，哨兵系统被开发出来了，哨兵可以在主发生故障后，自动进行故障转移，从从机里选出一台升级为主机，并持续监听着原来的主机，当原来的主机恢复后，会将其作为新主的从机。
### 集群架构
目前来说，高可用(主从复制、主从切换)redis集群有两种方案，一种是redis-sentinel，只有一个master，各实例数据保持一致；一种是redis-cluster，也叫分布式redis集群，可以有多个master，数据分片分布在这些master上。
  本文介绍基于docker和redis-sentinel的高可用redis集群搭建，大多数情况下，redis-sentinel也需要做高可用，这里先对redis搭建一主二从环境，另外需要3个redis-sentinel监控redis master。
很显然，只使用单个redis-sentinel进程来监控redis集群是不可靠的，由于redis-sentinel本身也有single-point-of-failure-problem(单点问题)，当出现问题时整个redis集群系统将无法按照预期的方式切换主从。官方推荐：一个健康的集群部署，至少需要3个Sentinel实例。另外，redis-sentinel只需要配置监控redis master，而集群之间可以通过master相互通信。
实际可靠的redis集群拓扑图如下：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041143259228.jpg)

### redis-sentinel
redis-sentinel作为独立的服务，用于管理多个redis实例，该系统主要执行以下三个任务：
* 监控 (Monitor): 检查redis主、从实例是否正常运作
* 通知 (Notification): 监控的redis服务出现问题时，可通过API发送通知告警
* 自动故障迁移 (Automatic Failover): 当检测到redis主库不能正常工作时，redis-sentinel会开始做自动故障判断、迁移等操作，先是移除失效redis主服务，然后将其中一个从服务器升级为新的主服务器，并让失效主服务器的其他从服务器改为复制新的主服务器。当客户端试图连接失效的主服务器时，集群也会向客户端返回最新主服务器的地址，使得集群可以使用新的主服务器来代替失效服务器
## 添加哨兵模式
### 搭建三节点redis主从集群
在上面的基础上新建一个从服务器，分别将两个从服务器的name设置为redis-slave-1和redis-slave-2，其中redis-slave-2端口设置为6381，并且此时为了方便外围系统通过sentinel调用redis集群并且方便sentinel通过端口区分redis集群的机器，需要更改启动命令为如下：
启动主服务器
```bash
docker run -d --net host --name redis -v /hzero/repo/redis/redis-master.conf:/usr/local/redis.conf redis redis-server /usr/local/redis.conf
```
启动从服务器1
```bash
docker run -d --net host --name redis-slave-1 -v /hzero/repo/redis/redis-slave-1.conf:/usr/local/redis.conf redis redis-server /usr/local/redis.conf
```
启动从服务器2
```bash
docker run -d --net host --name redis-slave-2 -v /hzero/repo/redis/redis-slave-2.conf:/usr/local/redis.conf redis redis-server /usr/local/redis.conf
```
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041198144097.jpg)

进入主服务器可以看到：
主服务器正确识别到两个slave：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041137001865.jpg)

两个slave也正确的挂载到了master上：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041137220816.jpg)

### 获取并修改sentinel配置
通过wget命令获取sentinel的配置文件：
```bash
wget http://download.redis.io/redis-stable/sentinel.conf
```
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041199997668.jpg)

修改配置文件以下几项
```conf

# 修改日志文件的路径
logfile "/var/log/redis/sentinel.log"

# 修改监控的主redis服务器
# 最后一个2表示，两台机器判定主被动下线后，就进行failover(故障转移)
sentinel monitor mymaster 35.236.172.131 6379 2
```
### 启动容器并查看日志状态
通过以下命令启动容器：
```bash
docker run -it --name sentinel --net host -v /hzero/repo/redis/sentinel.conf:/usr/local/etc/redis/sentinel.conf -d redis redis-sentinel /usr/local/etc/redis/sentinel.conf
```
其中/hzero/repo/redis/sentinel.conf为下载并修改的配置文件的位置，/usr/local/etc/redis/sentinel.conf是docker容器中配置文件的位置，redis-sentinel /usr/local/etc/redis/sentinel.conf表示在容器启动时使用配置文件启动哨兵
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041194762289.jpg)

通过日志可以查看到，两个slave通过设置的master的监控正确的被sentinel识别到。
### 测试sentinel
重新开一个shell并且停掉master：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041187993663.jpg)

通过sentinel的日志可以看到哨兵判断6379端口的master下线，并且开始投票选举新的master，选举完毕后将主服务器切换到了新的master上，然后将其他的还在线的从服务器挂载到新的master上面去。

此时登陆新的master查看集群信息：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041137983846.jpg)

可以看到原本的slave-2角色已经由slave转换为master，原本的slave-1也已经挂载到了新的master上：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041138164977.jpg)

此时重启原来的master，它并不会重新作为master运行，在哨兵的控制下他会以slave的角色挂载到新的master上。
查看sentinel日志可以看到：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041199018578.jpg)

再进入新的主服务器可以查看原来的master已经作为slave角色挂载：
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041138554218.jpg)

进入原来的主服务器也可以查看角色已经切换为了slave
![](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/01/01/17041138708495.jpg)

至此，哨兵模式下的redis集群已经搭建完毕！
# springboot整合redis哨兵模式
## 普通springboot项目整合哨兵
### 配置
pom.xml 添加Redis依赖
```xml
<!-- redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
application.yml 添加Redis配置
```yml
  redis:
    ##哨兵
    sentinel:
      master: mymaster
      nodes:
      - 172.21.0.211:7005
      - 172.21.0.211:7006
      - 172.21.0.211:7007
    ##
    jedis:
      pool:
        ### 连接池最大连接数（使用负值表示没有限制） 
        max-active: 9
        ### 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-wait: -1
        ### 连接池中的最大空闲连接 
        max-idle: 9
        ### 连接池中的最小空闲连接 
        min-idle: 0
    ### Redis数据库索引(默认为0) 
    database: 0
    ### 连接超时时间（毫秒） 
    timeout: 60000
```




