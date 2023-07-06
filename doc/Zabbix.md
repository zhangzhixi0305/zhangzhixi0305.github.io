视频地址：https://www.bilibili.com/video/BV1iY411E7Ee

Zabbix中文文档：https://www.zabbix.com/documentation/current/zh

资料地址：

## 一、Zabbix入门

###  1.1、Zabbix概述

　　Zabbix是一款能够监控各种网络参数以及服务器健康性和完整性的软件。

Zabbix使用灵活的通知机制，允许用户为几乎任何事件配置基于邮件的告警，这样可以快速反馈服务器的问题。基于已存储的数据，Zabbix提供了出色的报告和数据可视化功能。

Zabbix支持轮询和被动捕获。基于Web的前端页面确保您的网络状态和服务器健康状况可以从任何地方进行评估。在经过适当的配置后，Zabbix可以在监控IT基础设施方面发挥重要作用。

无论是对于拥有少量服务器的小型组织，还是拥有大量服务器的大型公司而言，同样适用。

### 1.2、Zabbix基础架构

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061822893.png)

核心组件主要是**Agent**和**Server**，其中Agent主要负责采集数据并通过主动或者被动的方式采集数据发送到Server/Proxy，除此之外，为了扩展监控项，Agent还支持执行自定义脚本。、

Server主要负责接收Agent发送的监控信息，并进行汇总存储，触发告警等。

ZabbixServer将收集的监控数据存储到Zabbix Database中。

Zabbix Database支持常用的关系型数据库，如果MySQL、PostgreSQL、Oracle等，默认是MySQL，并提供Zabbix Web页面（PHP编写）数据查询。

### 1.3、Zabbix和Prometheus

关于Prometheus资料，可以看我的另外一篇博客：https://www.cnblogs.com/zhangzhixi/p/17387191.html

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061825776.png)

综合对比：

> 从开发语言上看：

　　为了应对高并发和快速迭代的需求，监控系统的开发语言已经慢慢从C语言转移到Go。

不得不说，Go凭借简洁的语法和优雅的并发，在Java占据业务开发，C占领底层开发的情况下，准确定位中间件开发需求，在当前开源中间件产品中被广泛应用。

> 从系统成熟度上看：

　　Zabbix是老牌的监控系统：Zabbix是在1998年就出现的，系统功能比较稳定，成熟度较高。

而Prometheus是最近几年才诞生的，虽然功能还在不断迭代更新，但站在巨人的肩膀之上，在架构设计上借鉴了很多老牌监控系统的经验；

> 从数据存储方面来看：

Zabbix采用关系数据库保存，这极大限制了Zabbix采集的性能，而Prometheus自研一套高性能的时序数据库，在V3版本可以达到每秒千万级别的数据存储，通过对接第三方时序数据库扩展历史数据的存储；

> 从配置复杂度上看：

　　Prometheus只有一个核心server组件，一条命令便可以启动，相比而言，其他系统配置相对麻烦；

> 从社区活跃度上看：

　　目前Zabbix比较活跃，但基本都是国内的公司参与，Prometheus在这方面占据绝对优势，社区活跃度虽然不如，但是受到CNCF的支持，后期的发展值得期待；

> 从容器支持角度看：

　　由于Zabbix出现得比较早，当时容器还没有诞生，自然对容器的支持也比较差。而Prometheus的动态发现机制，不仅可以支持swarm原生集群，还支持Kubernetes容器集群的监控，是目前容器监控最好解决方案。

> 结论：

　　如果监控的是物理机，用Zabbix，Zabbix在传统监控系统中，尤其是在服务器相关监控方面，占据绝对优势。甚至环境变动不会很频繁的情况下，Zabbix也会比Prometheus好使；但如果是云环境的话，除非是Zabbix玩的非常溜，可以做各种定制，否则还是Prometheus吧，毕竟人家就是干这个的。Prometheus开始成为主导及容器监控方面的标配，并且在未来可见的时间内被广泛应用。如果是刚刚要上监控系统的话，不用犹豫了，
Prometheus准没错

##  二、Zabbix部署

### 2.1、集群规划

我这里准备的是三台服务器(均是Centos7)，一台做Server，两台agent，这里的要求是两台主机能够互通。具体步骤如下：

这里我是通过：tailscale，做的异地组网(局域网)，关于异地组网可以看我的博客：https://www.cnblogs.com/zhangzhixi/p/17490876.html

因为还需要使用到MySQL做持久化，我这里使用的MySQL版本是8.0.18，安装步骤参考：https://www.cnblogs.com/zhangzhixi/p/15849992.html

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826888.png)

### 2.2、环境搭建

#### 2.2.1、[关闭防火墙]

如果你知道应该开放哪些端口，可以单独设置开启指定端口，这里为了方便，就直接将防火墙关闭了：

```
# 关闭防火墙
systemctl stop firewalld.service
# 禁止开机自启
systemctl disable firewalld.service
```

#### 2.2.2、关闭SELinux

临时关闭：setenforce 0

永久关闭：

```
vim /etc/selinux/config
SELINUX=disabled
然后reboot	
```

#### 2.2.3、配置3台节点的Zabbix yum源

1、安装 zabbix 的软件仓库配置包：

```
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
```

2、安装 Software Collections 仓库

```
yum -y install centos-release-scl
```

3、替换为阿里云镜像

```
sed -i 's/http:\/\/repo.zabbix.com/https:\/\/mirrors.aliyun.com\/zabbix/g' /etc/yum.repos.d/zabbix.repo
```

4、打开/etc/yum.repos.d/zabbix.repo文件，启用zabbix-web仓库

```
vim /etc/yum.repos.d/zabbix.repo
修改
[zabbix-frontend]
enabled=1
```

### 2.3、安装Zabbix　　

#### 2.3.1、yum安装

在三台主机，分别执行以下命令：

> server(100.127.184.112)

```
yum -y install zabbix-server-mysql zabbix-agent zabbix-web-mysql-scl zabbix-apache-conf-scl
```

默认安装地址：/usr/share/zabbix

Zabbix日志路径：/var/log/zabbix/zabbix_server.log

> agent(100.99.171.50)、agent(100.87.7.20)

```
yum -y install zabbix-agent
```

#### 2.3.2、Zabbix数据库创建

在server那台服务器执行以下命令：

1、创建zabbix数据库

```
mysql -u root -pzhixi158 -e"create database zabbix character set utf8 collate utf8_bin"
```

2、导入zabbix建表语句

```
zcat /usr/share/doc/zabbix-server-mysql-5.0.36/create.sql.gz | mysql -u root -pzhixi158 zabbix
```

3、解决mysql8.x身份认证异常问题（在下面配置Zabbix的时候踩过坑了，这里可以先解决）

参考：https://www.cnblogs.com/zhangzhixi/p/16989906.html#_label1_1　　

#### 2.3.3、配置Zabbix

> server(100.127.184.112)

```
vim /etc/zabbix/zabbix_server.conf
#主机ip修改
DBHost=100.127.184.112
#数据库名称修改
DBName=zabbix
#数据库用户名及密码
DBUser=root
DBPassword=123456
```

> agent(100.127.184.112)、agent(100.99.171.50)、agent(100.87.7.20)

```
vim /etc/zabbix/zabbix_agentd.conf
```

三台主机分别修改Server为zabbix-server的ip，然后注释ServerActive、Hostname

```
Server=100.127.184.112
#ServerActive=127.0.0.1
#Hostname=Zabbix server
```

#### 2.3.4、修改Web时区

修改Server端即可：

```
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
php_value[date.timezone] = Asia/Shanghai
```

#### 2.3.5、[修改Httpd端口]

　　因为通过yum安装，使用的是ApacheHttpd做web展示，Httpd默认是80端口，如果想要修改zabbix访问端口的话，修改配置文件即可：

httpd日志路径：/var/log/httpd.log

```
vim /etc/httpd/conf/http.conf
Listen 8060
```

#### 2.3.6、启动Zabbix　

以下列出了四个命令，按需执行，这里我就只执行启动命令，分别是：启动(start)、停止(stop)、开机自启(enable)、禁用开机自启(disable)

> server(100.127.184.112)

```
systemctl start zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl stop zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl disable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```

> agent(100.99.171.50)、agent(100.87.7.20)

```
systemctl start zabbix-agent
systemctl stop zabbix-agent
systemctl enable zabbix-agent
systemctl disable zabbix-agent
```

### 2.4、访问Zabbix

访问地址：http://100.127.184.112:8060/zabbix

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826179.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826849.png)

配置数据库连接信息(也就是在2.3.3中设置的)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826339.png)

然后一直Next，直到登录界面

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826037.png)

登录Zabbix，默认用户名：**Admin** 密码：**zabbix**

修改语言为中文：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826624.png)

## 三、Zabbix使用

### 3.1、Zabbix术语

主机（Host）
　　一台你想监控的网络设备，用IP或域名表示。
监控项（Item）
　　你想要接收的主机的特定数据，一个度量数据。
触发器（Trigger）
　　一个被用于定义问题阈值和“评估”监控项接收到的数据的逻辑表达式。
动作（Action）
　　一个对事件做出反应的预定义的操作，比如邮件通知。

### 3.2、Zabbix实战（***）

#### 3.2.1、创建及配置主机（Host）

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826044.png)

#### 3.2.2、创建监控项（Item）

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826228.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826151.png)

**查看添加的监控项数据**

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826011.png)

####  3.2.3：创建触发器（Trigger）

配置-->主机-->触发器

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826811.png)

编辑触发器

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826017.png)

查看创建的触发器

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826017.png)

#### 3.2.4：创建动作（Action）

配置-->动作-->创建动作

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826898.png)

查看创建的动作

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826671.png)

#### 3.2.5：配置邮箱

QQ邮箱授权码获取：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826856.png)

管理-->报警媒介类型-->Email

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826496.png)

测试Email发送

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826900.png)

#### 3.2.6：配置事件触发器邮件发送收件人

管理-->用户-->报警媒介

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826922.png)

#### 3.2.7：测试

根据上图的监控项所示，现在连接数是293，我们这边启用几个Tomcat进行测试

要确保：监控项、动作、报警媒介类型都是已启用的状态~

------

 **服务器内连接数大于295，触发了邮件告警，并在首页显示**

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061826044.png)

Zabbix内展示

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827698.png)

对于首页仪表盘的问题，我们想让它消失通常是

1、点击确认问题

2、去排查解决问题

3、如果解决问题后进程数小于了295，那么这个告警会自动消失

4、如果直接关闭了问题，等下一次，也就是5s后，那么就会继续发送邮件（没解决问题）

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827010.png)

### 3.3、Zabbix模板

　　模板是可以方便地应用于多个主机的一组实体。

实体可以是监控项、触发器、图形、应用、web场景等。由于生产上的许多主机是相同或类似的，所以，为一个主机创建的一组实体（项目，触发器，图形，...）可能对其它主机也适用。

当然，你可以将它们复制到每个新的主机上，但需要费很大功夫。相反，使用模板，可以将它们复制到一个模板，然后根据需要将模板应用于尽可能多的主机。

因此，使用模板是减少工作量并简化Zabbix配置的好方法。另外，使用模板还有一个好处是当所有主机都需要更改时，只需要在模板上更改某些内容将会将更改应用到所有链接的主机。

　　**比如上面创建的监控项、触发器，其他两台服务器也都需要相同的配置，那么就可以使用模板来简化操作**

> 1、创建模板

配置-->模板-->创建模板

这里的群组是开始演示的那三台主机，将它们设置为了一个群组。

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827510.png)

> 2、创建监控项

这里我们来点不一样的，前面的例子是，服务器内连接数大于295，就触发告警，这里演示一个其他的：**这三台服务器中Tomcat进程数小于1，就触发告警**

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827878.png)

添加Tomcat监控

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827529.png)

> 3、配置触发器

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827745.png)

点击触发器后，同样的，在右上角点击创建触发器

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827244.png)

> 3、配置动作（发送邮件的动作）

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827546.png)

更新后的动作界面

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827786.png)

> 4、为主机应用模板

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827214.png)

同理，这三台主机都要添加，添加后的效果如下

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827665.png)

>  5、测试

　　在创建触发器的时候，我已经在这三台服务器上面都启动了一个Tomcat进程，现在我们测试下，依次关闭Tomcat，是否会触发告警。

**100.127.184.112：**

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827535.png)

**100.99.171.50：** 

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827447.png)

**100.87.7.20：**

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827849.png)

测试完成后，可以通过：配置-->动作-->移除触发器/停用该动作。

## 四、Zabbix整合Grafana

### 4.1、安装Grafana

　　grafana 是一款采用 Go 语言编写的开源应用，主要用于大规模指标数据的可视化展现，是网络架构和应用分析中最流行的时序数据展示工具，目前已经支持绝大部分常用的时序数据库。

Grafana下载地址：https://grafana.com/grafana/download

Grafana监控面板下载：https://grafana.com/grafana/dashboards/

在前面讲解中，已经安装了软件：**Zabbix**，负责收集本机数据，这时候Grafana的作用就是将这些数据做一个展示的功能！

> 1、安装

　　将软件包grafana-8.1.2.linux-amd64.tar.gz上传到/usr/local/grafana下，记得新建grafana文件夹

```
tar -zxvf grafana-enterprise-8.1.2.linux-amd64.tar.gz
cd grafana-8.1.2
nohup ./bin/grafana-server web > ./grafana.log 2>&1 &
```

访问：[localhost:3000](localhost:3000)，即可看到Grafana监控界面，默认用户名密码都是admin

> 2、加入开机自启：

```
#!/bin/bash
cd /usr/local/grafana/grafana-8.1.2
nohup ./bin/grafana-server web > ./grafana.log 2>&1 &
```

------

```
echo "/usr/local/grafana/grafana-8.1.2/start_grafana.sh"  >> /etc/rc.d/rc.local
```

### 4.2、集成Zabbix

> 1、配置数据源

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827005.png)

因为Grafana默认没有集成Zabbix数据源，所以需要安装插件。

**这里需要注意的是**，一定要选择合适的版本，如下图，我们这里安装的Grafana版本是：8.1.2，支持的最高Zabbix插件版本是：4.2.6

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827401.png)![img](https://img2023.cnblogs.com/blog/2126720/202307/2126720-20230706153917524-1493598195.png)

还需要注意的是，这里有两种安装方式，一种是通过命令安装，一种是通过下载插件，手动放在plugins目录下：

通过：Grafana配置文件(/usr/local/grafana/grafana-8.1.2/conf/defaults.ini)，可以知道，默认插件位置是：[plugins = data/plugins]

**第一种安装方式：**

　　(需要加版本号，并指定安装路径)：如果不加安装路径，那么默认的安装路径就是：/var/lib/grafana/plugins，那这其实就不对，这里是需要注意的。

```
cd /usr/local/grafana/grafana-8.1.2/bin
./grafana-cli --pluginsDir=/usr/local/grafana/grafana-8.1.2/data/plugins plugins install alexanderzobnin-zabbix-app 4.2.6
```

安装完成后重启Grafana

**第二种安装方式：**

　　下载插件，然后放到：/usr/local/grafana/grafana-8.1.2/data/plugins，下然后重启Grafana即可

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827977.png)



> 2、启用插件

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827849.png)

> 3、配置数据源

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827026.png)

> 4、添加Grafana展示面板

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827638.png)

 展示数据结果如下所示：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827035.png)

##  五、Zabbix整合睿象云告警平台

睿象云官网：https://caweb.aiops.com/

集成Zabbix

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827059.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827388.png)

> 1、查看Zabbix脚本目录

```
vim /etc/zabbix/zabbix_server.conf
```

搜索：AlertScriptsPath

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061827574.png)

进入脚本目录：

```
cd /usr/lib/zabbix/alertscripts
```

> 2、获取 Cloud Alert Agent 包

```
wget https://download.aiops.com/ca_agent/zabbix/ca_zabbix_release-4.0.3.tar.gz
```

> 3、解压，安装　　

```
tar -xzf ca_zabbix_release-4.0.3.tar.gz
cd cloudalert/bin
bash install.sh bf4c341497c647e3b5e657e542da46e7
```

Zabbix管理地址：你自己的Zabbix访问地址

默认用户名密码是：Admin zabbix

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828588.png)

成功安装：

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828395.png)

这里就是我们安装完成睿象云后，自动生成的报警方式了

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828755.png)

> 4、分派策略：告警信息发送给谁

如果Zabbix触发告警了，这时候就需要让睿象云来做告警了。

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828139.png)

> 5、通知策略

简单来说就是什么错误级别，还有消息的发送方式等的设置

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828587.png)

> 6、测试

现在测试关闭一台Tomcat，看下Zabbix和Grafana界面的消息

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828689.png)

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828984.png)

 短信和邮件

![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828882.png)![img](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202307061828489.png)