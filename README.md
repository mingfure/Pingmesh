# Pingmesh
> 大型数据中心网络延迟测量和分析的系统
## 一、 原理

> 在所有节点（服务器）上启动TCP或HTTP ping,以提供最大网络延迟测量覆盖率，从而实现一个用于大型数据中心网络延迟测量和分析的系统。

![img](https://cmhdb-1251602287.cos.ap-guangzhou.myqcloud.com/1602746867557.png)

## 二、 Pingmesh简单框架

该Pingmesh的简单框架主要分为可视化、Pingmesh Controller、Pingmesh Agent三部分，可视化提供完整的实时网络时延展示，Pingmesh Controller负责管理Pinglist，Pingmesh Agent在每台服务器上都需要安装，负责端到端网络测试发起。

1. 原始项目来源：https://github.com/aprilmadaha/pingmesh

2. 简单框架示意图

![img](https://cmhdb-1251602287.cos.ap-guangzhou.myqcloud.com/1602746900271.png)

3. 文件结构
   1. 语言：Go
   2. 数据库：MariaDB
   3. 探测器：fping
   4. UI：Python flask

![img](https://cmhdb-1251602287.cos.ap-guangzhou.myqcloud.com/1602746922624.png)

根据上图可以看到：

1. 服务器节点上的Pingmesh Agent（由调用fping的client.go实现）运行后将节点IP上传至Pingmesh Controller（由GetHostIP.go和GetResult.go共同实现）；
2. Controller将IP写入数据库；

3. Controller调用库中存储的网络拓扑，并生成pinglist；

4. Agent根据pinglist调用fping进行端到端测试，并将结果上传至Controller；

5. Controller将fping结果写入数据库；

6. 可视化部分（基于flask实现的Pingmesh.py）调用数据库并前端展示；

## 三、 安装编译
> 在该框架中，Controller负责IP收集拓扑生成，提供pinglist给Agent。因此Agent需要知道Controller的IP进行通讯。

1. 服务端（Controller+DSA）

   1.1 环境：linux

   1.2 软件：golang/MariaDB

2. 节点(服务器)

   2.1 环境：Linux

   2.2 软件：fping

3. UI
   
### Controller/DSA配置
```shell
# 1. 安装golang/mariadb,设置防火墙
yum install -y golang
sudo yum install mariadb-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
mysql_secure_installation //初始化mariadb
sudo systemctl stop firewalld.service
setenforce 0

# 2. 导入数据库表(pingmesh.sql)

# 3. 编译

go build pingmesh-s-v1.1-GetResult.go
go build pingmesh-s-v1.1-GetHostIp.go

# 4. 运行
nohup ./pingmesh-s-v1.1-GetResult > output.log 2>&1 &
nohup ./pingmesh-s-v1.1-GetHostIp > output.log 2>&1 &

```

### Agent配置
```shell
# 1. 安装fping
yum install epel-release -y
yum install fping -y

# 2.编译Agent
go build pingmesh-c-v1.1.go

# 3.运行
nohup ./pingmesh-c-v1.1 > output.log 2>&1 &

```

### 可视化设置
```shell
# 0. 调整参数pingmesh.py
conn = pymysql.connect(
host='172.19.129.11',
user='root',
password='123456',
db='ping',
charset='utf8'
)
# 1. 安装flask/pymysql
pip install flask
pip install pymysql

# 2.运行项目
python pingmesh.py
后台运行：nohup python pingmesh.py > pingmeshpy.log 2>&1 &

# 3.结果显示
http://127.0.0.1:9000
```
