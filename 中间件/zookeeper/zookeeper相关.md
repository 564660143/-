# zookeeper简介

- zookeeper是一个开源的分布式协调服务，是由雅虎创建的，基于google chubby

- zookeeper是什么 : 分布式数据一致性的解决方案

- zookeeper能做什么

  > - 数据的发布/订阅（配置中心:disconf）, 类似MQ
  > - 负载均衡（dubbo利用了zookeeper机制实现负载均衡）
  > - 命名服务
  > - master选举(kafka、hadoop、hbase)
  > - 分布式队列
  > - 分布式锁
  > - 统一配置文件管理
  > - 集群管理, 及群众保证数据的强一致性
  > - ...

- zookeeper的特性

  > - **顺序一致性 :** 从同一个客户端发起的事务请求，最终会严格按照顺序被应用到zookeeper中
  > - **原子性 :** 所有的事务请求的处理结果在整个集群中的所有机器上的应用情况是一致的，也就是说，要么整个集群中的所有机器都成功应用了某一事务、要么全都不应用
  > - **可靠性 :** 一旦服务器成功应用了某一个事务数据，并且对客户端做了响应，那么这个数据在整个集群中一定是同步并且保留下来的
  > - **实时性 :** 一旦一个事务被成功应用，客户端就能够立即从服务器端读取到事务变更后的最新数据状态；（zookeeper仅仅保证在一定时间内，近实时）
  > - **单一视图 :**客户端连接集群中的任一zk节点, 数据都是一致的

# zookeeper安装

### 安装步骤

1. 下载zookeeper安装包

   > http://apache.fayea.com/zookeeper/stable/zookeeper-3.4.10.tar.gz

2. 解压zookeeper

   > tar -zxvf zookeeper-3.4.10.tar.gz

3. cd 到 ZK_HOME/conf  , copy一份zoo.cfg

   > cp  zoo_sample.cfg  zoo.cfg

4. 启动服务端

   > sh zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}

5. 客户端连接

   > sh zkCli.sh -server  ip:port

# 集群环境

## 概述

zookeeper集群中包含三种角色 : leader/follower/observer

### observer 

> observer 是一种特殊的zookeeper节点。可以帮助解决zookeeper的扩展性（如果大量客户端访问我们zookeeper集群，需要增加zookeeper集群机器数量。从而增加zookeeper集群的性能。 导致zookeeper写性能下降， zookeeper的数据变更需要半数以上服务器投票通过。造成网络消耗增加投票成本）
>
> 1. observer不参与投票。 只接收投票结果。
>
> 2. 不属于zookeeper的关键部位。

observer 节点配置:

```properties
#1.	在zoo.cfg里面增加
peerType=observer
#2.server配置的最后面添加":observer"
server.1=192.168.72.135:2888:3181:observer
```

## 集群环境搭建

1. 修改配置文件, server.id=host:port1:port2

   > **id :** 取值范围1-255; 用id来标识该机器在集群中的机器序号
   >
   > **port1:** 表示follower节点与leader节点交换信息的端口号
   >
   > **port2 :** 表示leader选举的端口, 如果leader节点挂掉了, 需要一个端口来重新选举

   我本地虚拟机实际配置如下:

   ```properties
   server.1=192.168.72.135:2888:3181
   server.2=192.168.72.136:2888:3181
   server.3=192.168.72.137:2888:3181
   ```

2. 创建myid

   > 在每一个服务器的dataDir目录下创建一个myid的文件，文件就一行数据，数据内容是每台机器对应的server ID的数字

3. 启动zookeeper

# zoo.cfg配置文件分析

```properties
# zookeeper中最小的时间单位长度 （ms）, 用于计算的时间单元
tickTime=2000  
# follower节点启动后与leader节点完成数据同步的时间, 10表示10 * tickTime
initLimit=10  
# leader节点和follower节点进行心跳检测的最大延时时间, 同样5表示5 * tickTime
syncLimit=5 
# 必须配置,表示zookeeper服务器存储快照文件的目录
dataDir=/data/zookeeper  
# 配置 zookeeper事务日志的存储路径，默认指定在dataDir目录下
dataLogDir=/data/zookeeper
# 表示客户端和服务端建立连接的端口号, 默认2181
clientPort=2181
```

# zookeeper基本数据模型

- 是一个树形结构, 类似于前端开发中的tree.js
- 每一个节点都称之为znode, 它可以有子节点, 也可以有数据
- 每个节点分为临时节点和永久节点, 临时节点在客户端断开后消失
- 每个zk节点都有各自的版本号, 可以通过命令行来显示节点信息
- 每当节点数据发生变化, 那么该节点的版本号会累加(乐观锁)
- 删除/修改过时节点, 版本号不匹配则会报错
- 每个zk节点存储的数据不宜过大, 一般几K即可
- 节点可以设置权限acl, 可以通过权限来控制用户的访问

# zookeeper常用命令

- sh zkCli.sh -server  ip:port : 客户端连接

- ls : 类似于Linux中的ls

- ls2 : 展示详细节点信息

- stat : 节点状态信息

- get : 读取节点数据

  ```properties
  # 创建节点之后, zk为节点分配的节点id
  cZxid = 0x0
  # 节点创建时间
  ctime = Thu Jan 01 00:00:00 GMT 1970
  # 修改后的节点id
  mZxid = 0x0
  # 节点修改时间
  mtime = Thu Jan 01 00:00:00 GMT 1970
  # 子节点id
  pZxid = 0x0
  # 子节点的version, 子节点发生变化时version会更改
  cversion = -1
  # 当前节点数据version, 数据发生变化时version会更改
  dataVersion = 0
  # 权限version, 权限发生变化时version会更改
  aclVersion = 0
  # 
  ephemeralOwner = 0x0
  # 数据长度
  dataLength = 0
  # 子节点个数
  numChildren = 1
  ```

## Session相关

- 客户端与服务端之间的连接存在会话
- 每个会话都可以设置一个超时时间
- 心跳结束, session就过期, session过期, 则临时节点znode会被删除
- 心跳机制 : 客户端向服务端的ping包请求

### 命令

- 创建节点 : create -\[s\]\[e\]  \[nodeName][nodeData] 

  > -s : 表示创建一个永久有序节点
  >
  > -e : 表示创建一个临时节点
  >
  > 默认创建的是永久无序节点

- 修改节点值 : set \[nodeName] [new Data] \[version]

  > - 使用set nodeName newData可以直接进行更新
  > - 如果添加了version的话, 表示时乐观锁模式, 如果version版本不一致的话, 就会更新失败

- 删除节点 : delete \[path]  [version]

  > version和修改节点一样, 用于乐观锁

## zookeeper四字命令

- zk可以通过它自身提供的简写命令和服务器进行交互
- 需要使用到nc命令, 安装 : yum install nc
- echo [command]|nc \[ip] [port]
- 需要在zoo.cfg增加 "4lw.commands.whitelist=*"来增加四字命令的支持

**常用四字命令:**

- stat : 查看zk的状态信息, 以及mode
- ruok : 查看当前zkserver是否启动, 返回imok
- dump : 列出未经处理的会话和临时节点
- conf : 查看服务器配置信息
- cons : 展示连接到服务器的客户端信息
- envi : 环境变量
- mntr : 监控zk的健康信息
- wchs : 展示watch的信息
- wchc&wchp : session与watch及path与watch的信息

# zookeeper特性

## watcher机制

- 针对每个节点的操作, 都会有一个监督者  -> watcher

- 当监控的某个对象(znode)发生了变化, 则触发watcher事件

- zk中的watcher是一次性的, 出发后立即销毁

- 父节点, 子节点增删改都能触发watcher

- 针对不同类型的操作, 触发的watcher事件也不同 :

  > - (子)节点创建事件
  > - (子)节点删除事件
  > - (子)节点数据变化事件

### 父节点watcher事件

- 通过get path [watch]设置watcher
- 父节点增删改操作触发watcher
- 子节点增删改触发watcher
- 创建父节点触发 : NodeCreated, 需要使用stat path watch设置
- 修改父节点触发 : NodeDataChanged
- 删除父节点 : NodeDelete

### 子节点watcher事件

- ls为父节点设置watcher, 创建子节点触发 : NodeChildrenChanged
- ls为父节点设置watcher, 删除子节点触发 : NodeChildrenChanged
- ls为父节点设置watcher, 修改子节点不触发事件 

### watcher使用场景

- 统一资源配置

## ACL权限控制

- 针对节点可以设置相关读写等权限, 目的为了保障数据安全性
- 权限permissions可以指定不同的权限范围以及角色
- getAcl : 获取某个节点的acl权限信息
- setAcl : 设置某个节点的acl权限信息
- addauth : 输入认证授权信息, 注册(登录)时输入明文密码,zk中密码以加密方式存储

**ACL的构成 :**

zk的ACL通过[scheme​ : id : permissions]来构成权限列表

- scheme : 代表采用的某种权限机制

  > - world : world下只有一个id, 即只有一个用户, 也就是anyone, 组合写法 world:anyone[permissions]
  > - auth : 代表认证登录, 需要注册用户有权限就可以, 格式 : auth:user:password:[permissions]
  > - digest : 需要对密码加密才能访问, 格式: digest:username:BASE64(SHA1(password)):[permissions]
  > - super : 代表超级管理员, 拥有所有的权限

- id : 代表允许访问的用户

  > - 当设置ip为指定的ip地址, 此时限制ip进行访问

- permissions : 权限组合字符串(cdrwa)

  > - CREATE : 创建子节点
  > - DELETE : 删除子节点
  >
  > - READ : 获取节点/子节点
  > - WRITE : 设置节点数据
  > - ADMIN : 设置权限

### 命令

- world:anyone:cdrwa

  > - crdwa可以任意组合

- auth:user:pwd:cdrwa

  > digest:user:BASE64(SHA1(pwd)):cdrwa
  >
  > 添加用户(登录) : addauth digest user:pwd

- digest

- ip:[ip addr] cdrwa : 针对ip设置权限

- super超级管理员

  > 1. 修改zkServer.sh增加super管理员
  > 2. 重启zk Server 

### ACL使用场景

- 开发/测试环境分离
  - 生产环境上控制指定ip的服务可以访问相关节点, 防止混乱
