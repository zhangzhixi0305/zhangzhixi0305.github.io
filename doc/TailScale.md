## 一、概要

### VPN 是什么？

　　看到 VPN 第一反应应该是翻墙，但 VPN 最初应该也是最普遍的用途应该是用来做内网打通， 这也是其名字虚拟私有网络的用意，VPN 让你可以在公开的网络线路上建立一个私有的子网， 然后将所有接入的机器都分配一个私有的内网地址，让他们可以通过 VPN 的私有网络互联。

### TailScale 是什么

　　云上VPN服务商，提供VPN的一种：mesh VPN；一种能让节点间直接互联，每一个节点都会存储其他所有节点的信息，并且和其他所有的节点都建立 TLS加密连接。

对个人用户免费，支持100设备；

> TailScale 能做到什么

　　只要你的机器可以连到公网，tailscale 可以让所有的机器连接到同一个私有子网内。 你可以像在同一个局域网里那样，随时随地的连接你的任意设备，场景：文件传输、远程开发、代理

## 二、使用

官网：https://tailscale.com/

tailscale管理端：https://login.tailscale.com/admin/machines

以下演示我是通过微软账户进行登录的；

### 1、注册

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619154240448-2086642219.png)

 登陆后进入添加设备页面，左边是提供添加设备方法，右边是你添加成功后会出现你的设备清单

###  2、下载安装

下载地址：https://tailscale.com/download/windows

官网给出了包含五种设备的安装以及连接方法，下面我就演示常用的三种：本机Windows、逍遥模拟器（Android）、Linux（阿里云）

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619154500535-1797140364.png)

### 3、 Windows

下载安装包进行安装即可。

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619154655184-642553814.png)

在电脑任务管理器会有tailscale的图标，点击登录即可，然后就可以看到你的这个主机了。

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619154750308-291613839.png)

进入管理界面：然后设置主机名，以及禁止秘钥过期：

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619155201975-942053275.png)

**禁用DNS**

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230626171527591-2028400983.png)

### 4、Linux

国内的网络下载好大约十分钟，因为国内的网络无法访问：https://pkgs.tailscale.com，所以这里就使用第二种方式进行下载安装。

```
curl -fsSL https://tailscale.com/install.sh | sh
```

------

 ![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619155444917-2037924841.png)

> **设置DNS(***)：这步已经通过前面的设置DNS进行处理了，此处不再设置**

先说问题：

　　在Linux下启动tailscale，不能ping通外网，将默认将系统dns文件：/etc/resolv.conf进行替换了。然后关闭tailscale，dns配置又恢复之前的样子了；

如果是这样的话，我开启了tailscale后，我不能够访问外网，这软件是有点耦合性太高了~

参考博客：https://www.nuomiphp.com/t/639ede9003d3a871b64869a5.html

官网给出的解决办法：https://tailscale.com/kb/1188/linux-dns/#dhcp-dhclient-overwriting-etcresolvconf

```
$ sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
$ sudo systemctl restart systemd-resolved
$ sudo systemctl restart NetworkManager
$ sudo systemctl restart tailscaled
```

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230626170443119-1437583868.png)

> tailscale帮助命令

```
[root@zhixi ~]# sudo tailscale --help
使用说明
	tailscale [标志] <子命令> [命令标志]

子命令：
	up 连接到Tailscale，如有需要则进行登录
	down 从Tailscale断开连接
	set 更改指定偏好设置
	login 登录到Tailscale账户
	logout 从Tailscale断开，并使当前节点密钥过期
	switch 切换到另一个Tailscale账户
	configure [ALPHA] 配置主机以启用更多的Tailscale功能
	netcheck 打印关于本地网络情况的分析
	ip 显示Tailscale IP地址
	status 显示tailscaled及其连接的状态
	ping 在Tailscale层对主机进行ping，查看其路由情况
	nc 连接到主机的一个端口，连接到stdin/stdout
	ssh SSH到一个Tailscale机器
	funnel 开启/关闭 Funnel服务
	serve 提供内容和本地服务器服务
	version 打印Tailscale版本
	web 运行一个用于控制Tailscale的web服务器
	file 发送或接收文件
	bugreport 打印一个可共享的标识符，以帮助诊断问题
	cert 获取TLS证书
	lock 管理tailnet锁
	licenses 获取开源许可证信息
```

> 通过systemctl进行操作

```
systemctl status tailscaled # 查看状态
systemctl start tailscaled # 启动
systemctl stop tailscaled # 停止
systemctl enable tailscaled # 开机自启
systemctl disable tailscaled # 禁用开机自启
```

### 5、 Android

安装应用，因为国内的原因，在应用商店下载不到tailscale的apk安装包，这里我已经下载好了：

蓝奏云：

　　https://wwxo.lanzouj.com/iXQTm0zixwpe

　　密码:dipu

------

 

软件安装完后，打开，也是通过微软账户进行登录，即可加入到网络：

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619162051666-765764026.png)

### 6、测试

我在本地Windows启动一个程序，然后测试在Linux、Android上面是否能够访问到应用程序。

**这里需要的局域网IP地址也就是控制台的ADDRESSES，注意区分**

本地程序访问地址：http://localhost:8080/user/selectAll/1/4

异地组网访问地址：http://100.103.222.102:8080/user/selectAll/1/4

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230619163348969-2053974731.png)

## 三、Nginx整合Tailscale做端口转发

> 使用场景：使本地应用能被通过公网IP地址进行访问

　　看以下图示，通过云服务器的公网IP，用户访问这个IP，通过Nginx转发，使用户访问到本地内网的应用，前提是需要**有云服务器和公网IP**

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230627095011211-2107219368.png)

> 操作步骤

- 1、在我本地启动了一个应用，访问地址是：[http://192.168.147.129:8080](http://192.168.147.129:8080/)，因为是部署在本地，通过互联网访问不了。
- 2、Nginx设置端口转发，比如互联网访问：[http://182.92.209.212:8023](http://182.92.209.212:8023/)，就让地址跳转到：[http://192.168.147.129:8080](http://192.168.147.129:8080/)
- 3、注意云服务放行安全组规则

Nginx配置：

```
# 设置NGINX代理转发，访问8023端口，转发到其他地址
server {
    listen       8023;
    server_name  localhost;

    location /{
		root   html;
        index  index.html index.htm;
		# 你的应用访问地址，使用Tailscale异地组网的IP
        proxy_pass http://100.99.171.50:8080;
    }
}
```

![img](https://img2023.cnblogs.com/blog/2126720/202306/2126720-20230627100022215-1055415330.png)