## Confluence搭建

参考链接地址：https://blog.csdn.net/qq_52716296/article/details/127284680

​                            https://www.cnblogs.com/ilanni/p/6204722.html

### 一、什么是Confluence

​ Confluence是一个专业的wiki程序。它是一个知识管理的工具，通过它可以实现团队成员之间的协作和知识共享。Confluence不是一个开源软件，非商业用途可以免费使用。
Confluence使用简单，但它强大的编辑和站点管理特征能够帮助团队成员之间共享信息，文档协作，集体讨论。
confluence是一个专业的企业知识管理与协同软件，可以用于构建企业wiki。通过它可以实现团队成员之间的协作和知识共享

> Confluence国内平替

pingcode：https://pingcode.com/

### 二、Confluence环境搭建

#### 环境准备

> 关闭selinux

```shell
setenforce 0 #临时关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config #永久关闭
```

> 更换阿里yum源

```shell
# 查看yum源信息
yum repolist
#1、备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup 
# 2、下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 3、清理缓存
yum clean all
# 4、重新生成缓存
yum makecache
```

#### Java环境安装

​ 略，安装1.7+版本即可，https://www.cnblogs.com/zhangzhixi/p/14399602.html#_label0_1

#### MySQL环境安装

​ 注意：**不能将MySQL 8.0与Confluence 7.1或更早版本一起使用**
，参考：https://confluence.atlassian.com/doc/database-setup-for-mysql-128747.html

​ MySQL5.7安装教程：https://www.cnblogs.com/zhangzhixi/p/14961507.html

> 创建数据库（等下安装Confluence需要用到这个库）

```sql
CREATE DATABASE confluence CHARACTER SET utf8 COLLATE utf8_bin;
FLUSH PRIVILEGES;
```

> 删除数据库

```sql
DROP DATABASE IF EXISTS confluence;
FLUSH PRIVILEGES;
```

#### 安装Confluence

> 1、下载软件安装包以及破解包

```shell
mkdir -p /usr/local/confluence

cd /usr/local/confluence

wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-5.6.6-x64.bin
```

破解包下载地址：https://u062.com/file/15323800-217465309，下载之后上传到/usr/local/confluence下即可

> 2、授权安装文件以及解压破解文件

```shell
chmod 777 atlassian-confluence-5.6.6-x64.bin
unzip confluence5.6.6-crack.zip
```

> 3、安装Confluence

```shell
./atlassian-confluence-5.6.6-x64.bin
```

![image-20230313150900510](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131509774.png)

安装目录: /opt/atlassian/confluence
主目录: /var/atlassian/application-data/confluence
HTTP端口: 8090

---

访问：Ip+端口

![image-20230313151145962](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131511120.png)

复制下这个ServerID，等下破解用：BB5A-JBX0-Z7B4-IGGT

![image-20230313151212700](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131512836.png)

#### 破解Confluence

> 1、关闭Confluence程序

```shell
service  confluence stop 
```

> 2、删除confluence安装生成的jar包

```shell
rm -fr /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-*
```

>
3、导入破解包里面的jar包到上述的lib目录解压破解包，把里面的atlassian-extras-3.2.jar、Confluence-5.6.6-language-pack-zh_CN.jar、mysql-connector-java-5.1.39-bin.jar三个jar文件复制到/opt/atlassian/confluence/confluence/WEB-INF/lib目录下

```shell
cp /usr/local/confluence/confluence5.6.6-crack/jar/* /opt/atlassian/confluence/confluence/WEB-INF/lib/
```

> 查看三个jar文件是否复制成功

```shell
ls|grep -E "atlassian-extras|Confluence-5.6.6-language-pack|mysql-connector-java"
```

> 4、启动Confluence程序

```shell
service  confluence start
```

> 5、Windows下执行破解程序

![image-20230313152339865](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131523047.png)

将key粘贴到程序中

![image-20230313152428598](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131524727.png)

切换到MySQL数据源

![image-20230313152555067](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131525144.png)

![image-20230313152655701](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131526824.png)

初始化数据库，稍微等会

![image-20230313153233590](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131532647.png)

选择他建议的即可，这里是第一次安装，没有备份数据

![image-20230313153509601](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131535738.png)

![image-20230313153607805](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131536932.png)

设置用户名和密码：admin 123456

![image-20230313153716252](https://typora-picgo-images-zhixi.oss-cn-beijing.aliyuncs.com/202303131537309.png)

#### Confluence关闭与启动命令

```shell
service  confluence stop
service  confluence start
service  confluence restart
```

### Confluence的卸载

> 1、删除主目录

```bash
rm -rf /opt/atlassian/
```

> 2、删除数据目录

```
rm -rf /var/atlassian/
```

> 3、删除用户

```
userdel -r confluence
```

> 4、删除数据库











