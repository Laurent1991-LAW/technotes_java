# Linux指令

# 1 前言

操作系统作用 : 从根本上看，用于与硬件交互（“磁盘你存个东西呗”）

## 采用Linux内核系统产品

红帽red hat : 安全性

centoOS : 课上使用

乌班图ubuntu : 云平台、并发要求高

中标麒麟 : 国家推崇

## 代表人物

Ken Tompson : B语言之父

Dennis Ritchie : C语言之父  <<C程序设计>> 谭浩强翻译

Linus Torvalds : 于1991年开发出Linux内核

## LINUX系统特点

- 系统开源且免费
- 对硬件要求低，系统总大小仅为 800 M / 1G左右
- 系统稳定性强
- 系统安全型更好佳（多用于军工企业><微软迄今未开源，用它作服务器系统，后果不敢想象）

# 2 Linux基本指令

注意 : “/” —— 为Linux根目录标志，比如从local目录下，无法直接 cd /src/ 进入src目录，因为“/”意味着从根目录root开始往下走，正确答案是  cd src 即可——相对路径写法

工作目录:	cd /usr/local/src/

## 基本文件夹操作

ifconfig/ip addr  	 检查IP地址

pwd                         检查当前的位置

tab键                        自动补齐(注意唯一性)

cd / 	返回根目录

cd ~ 	用户主目录，即root文件夹

cd . 	当前目录

cd ..	返回到上一级目录

cd /usr/ 	进入到usr目录

cd -   	撤销一步操作 / 返回上一个目录

cd 		直接回家

## 目录与文件

ls			查看目录下所有文件

ls -l 		详细格式，文件权限，时间

ll 			和 ls -l 作用相同

ls *.txt 		查看所有的txt类型文档

## 创建文件夹

mkdir 		创建目录

mkdir a 		创建 a目录

mkdir -p a/b 		创建 a目录，并在a目录里创建b目录

mkdir -m 777 c 			创建一个权限为777的C目录

rmdir  			删除目录（如果目录里有文件，则不能用此命令）

注: mkdir --help 可以查看帮助文档

## 创建/查看/编辑文件

vi / vim 		创建 / 查看文件，如 vim 1.txt

编辑模式：

- 按i，在光标前开始编辑
- 按a，在光标后开始编辑
- 按o，在当前行的下一行开始编辑
- 按u, 撤销之前的操作
- 底行模式：按  shift+：冒号。
- :q! 不保存退出 => 强制退出
- :wq 保存退出
- :/world 从当前光标处，向上查找world关键字
- :?world 从当前光标处，向后查找world关键字

## 删除指令

rm n.txt			提示性删除

rm -f n.txt     	免去提示，直接删除

rm -rf 目录名		直接递归删除目录下所有内容（带-r都与文件夹相关）

rm -rf * 			删除所有文件

rm -rf /* 		【慎用！删光所有能删的！】删除所有子目录所有文件及下属文件（"/"都是指根目录下）

## 复制与移动

cp 复制文件

cp nginx.conf n.txt

cp –R tomcat1 tomcat2                #复制整个目录

mv 修改文件名，移动文件

mv n.txt m.txt  修改文件名称

拓展:

逻辑删除 : rm，只是删除引用，一般删除都是该逻辑，所以有些删除仍可回复

物理删除 : 磁盘扔微波炉

# 3 Linux环境配置

将服务器部署到Linux系统中（蓝框内所有内容） :

​                  ![img](https://docimg1.docs.qq.com/image/JhrmA_ueW2DBHm6Qp2Wo9A.png?w=780&h=480)         

部署前提:

1. 消除数据库权限 ;
2. 关闭防火墙 ;



## 00 JAVA8安装及环境变量配置

将 jdk-8u212-linux-x64.tar.gz 上传至/root文件

```unix
# 将jdk解压到 /usr/local/ 目录, -C为指定解压目录
tar -xf jdk-8u212-linux-x64.tar.gz -C /usr/local/

# 切换到 /usr/local/ 目录, 显示列表, 查看解压缩的jdk目录
cd /usr/local

# 修改 /etc/profile 配置文件, 配置环境变量
vim /etc/profile

# 在文件末尾添加以下内容:
export JAVA_HOME=/usr/local/jdk1.8.0_212
export PATH=$JAVA_HOME/bin:$PATH

# 使环境变量立即生效
source /etc/profile

# 验证
java -version
```



#### rocketMQ安装及环境变量配置

```
# 下载rocketMQ二进制zip文件
wget --no-check-certificate https://dlcdn.apache.org/rocketmq/4.9.2/rocketmq-all-4.9.2-bin-release.zip

# 解压缩
unzip rocketmq-all-4.9.2-bin-release.zip -d /usr/local/

# 修改一下文件夹名，改成 rocketmq 方便使用
mv /usr/local/rocketmq-4.9.2 /usr/local/rocketmq

vim /etc/profile

# 在文件末尾添加以下内容:
export ROCKETMQ_HOME=/usr/local/rocketmq
export PATH=$ROCKETMQ_HOME/bin:$PATH

source /etc/profile
```



##  01 数据库安装与权限修改

1. ping [www.baidu.com](http://www.baidu.com) 测试链接  ctrl+c停止
2. yum  install mariadb-server   ——安装mariadb数据库（yum近似应用商店）
	. yum  clean   all  	——若出错则全删再安装
4. 命令:

​                  ![img](https://docimg8.docs.qq.com/image/5D2SoWMszRkbbR73Cp4fsA.png?w=497&h=117)         

注意 : 启动之后设定开机自启

1. 数据库初始化操作 : mysql_secure_installation (设定密码，其他一律Y)
	. mysql -u root -p	（登录）
3. 更改权限 : 找到 mysql / user 表 => 将localhost改为%

​                  ![img](https://docimg6.docs.qq.com/image/Mjw5mSKJHuhoT0F4woXanw.png?w=599&h=386)         登录成功后，找到所有库show databases ;

​                  ![img](https://docimg10.docs.qq.com/image/mT-xtb5ZwKZeNKG0Qj-EtA.png?w=1110&h=264)         进入mysql库 ;

​                  ![img](https://docimg3.docs.qq.com/image/LeHbRNQGhhymSZW_T3sqPg.png?w=1107&h=383)         查询host/user字段 ;

​                  ![img](https://docimg1.docs.qq.com/image/6AnZLak930aQEblx_jjrqg.png?w=1111&h=503)         将host=localhost改为host = %

​                  ![img](https://docimg2.docs.qq.com/image/JByaV-amOOeTdlZd6Xn6ww.png?w=641&h=167)         刷新权限

注意 : Linux系统中的mysql必须以 ; 结尾

## 02 防火墙关闭

src目录下:

1. firewall-cmd --state  查看状态
	. systemctl disable firewalld.service		未来关闭防火墙的服务（查看状态依旧为running）
	. systemctl stop firewalld.service	关闭当前防火墙

至此，可以通过windows数据库远程连接上Linux数据库

​                  ![img](https://docimg3.docs.qq.com/image/Pf87fYnuGAKiBfrdh8K4qg.png?w=623&h=474)         

## 03 后端项目发布

### Linux端JDK安装

- 安装JDK :
- 将Linux版jdk安装包jdk-8u51-linux-x64.tar.gz上传到src目录下，解压 ;
- 删除压缩文件、将文件名改为jdk1.8
- src目录下 : java -version （验证是否安装成功）
- vim /etc/profile   => 配置环境变量

```
#设定jdk环境
export JAVA_HOME=/usr/local/src/jdk1.8
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib
```

###  

### 导出IDEA工程/部署/运行

- yml文件 : 检查后端IDEA设置的数据库账号、密码是否与Linux系统信息符合

​                  ![img](https://docimg7.docs.qq.com/image/xVRRhdQNxHXBxFdBQJikXA.png?w=481&h=214)         

- 项目POM窗口界面 => 右边maven工具栏中LifeStyle => build => 进行项目编译打包，打包后的文件在targets中获取

​                  ![img](https://docimg1.docs.qq.com/image/U0i-6wkeTrrigS3TjLLd9w.png?w=297&h=194)         

- Linux系统src工作空间目录下创建tomcats文件夹，将8091、8092拉入目录下
- 一般运行命令 :   java -jar 8091.jar

#  

需求说明 : java -jar 8091.jar 指令操作后，该终端被占用无法进行其他操作 ; 若当前终端关闭，整个tomcat服务器也将关闭，启动项与终端绑定，该方式非常不友好

解决方案 : Linux系统中提供了后台运行方式

后台运行语法 :

进入tomcats文件夹 => nohup java -jar 8091.jar => 8091.log &

（注意 : 必须得进入tomcats服务器再实行该命令，否则文件log文件会建在其他位置）

#  

### JAVA日志查看

- vim 8091.log		通过:wq退出，但该命令多用于修改文件
	 cat 8091.log		瞄一眼，但log经常非常长，所以使用tail比较好
	 tail -10 8091.log  		显示文件后10行
	 tail -f 8091.log  		动态查看

例如：

​	滚动动态查看日志   ==>   cat tail -f 2091.log

​        查看/etc/profile的前10行内容

​        \# head -n 10 /etc/profile



​        查看/etc/profile的最后5行内容，应该是：

​        \# tail  -n 5 /etc/profile



​        查看/etc/profile文件中第3000行开始，显示1000行的内容。即显示3000~3999行

​        \# cat filename | tail -n +3000 | head -n 1000



​        查看/etc/profile文件中第1000行到3000行的内容。

​        \# cat filename| head -n 3000 | tail -n +1000 



### 杀死进程

后台运行进程中的Java项目无法进入，因此无法通过ctrl + c进行直接关闭

输入指令

jps => 找到java进程的进程号 

或者

ps -ef  |  grep java   查找java程序的运行信息，包含PID

=> kill PID号 —— 一般类型杀死进程（不够强硬）

=> kill -15 PID号  —— 较为强硬的杀死

=> kill -9 PID号 —— 强制杀死，后果自负

## 04 安装部署Nginx

1. Nginx官网上下载最近Nginx的Linux版本 ; 
2. 将nginx-1.21.3.tar.gz压缩包放入src目录下 ;
3. 输入解压命令 :   tar -xvf nginx-1.21.3.tar.gz ;
4. 删除压缩包文件，将解压出的文件夹利用mv命令改名为nginx-source ;
5. 进入Nginx根目录下，依次输入以下三行命令 :

	/configure	=>		make	=>		make install

1. 利用whereis nginx找到nginx文件所在地（一般在local里与src平级），进入nginx/sbin，输入相关指令 :

启动命令:    ./nginx

重启命令:     ./nginx -s reload

关闭命令:      ./nginx -s stop

1. 修改Nginx.conf

```
# 配置图片代理  记得保存
	server {
		listen 80;
		server_name image.jt.com;
		location / {
			root  /usr/local/src/images;
		}
	}
	# 配置前端服务器 www.jt.com:80 dist/index.html
	server {
		listen 80;
		server_name www.jt.com;
		location / {
			root  dist;
			index index.html;
		}
	}
	#manage.jt.com:80 映射localhost:8091
	server {
		listen 80;
		server_name manage.jt.com;
		location / {
			proxy_pass http://tomcats;
		}
	}
	#配置后端集群 1.默认轮询 2.权重 weight 3.iphash策略
	upstream tomcats {
		#ip_hash;  weight=4;
		server 192.168.126.129:8091;
		server 192.168.126.129:8092;
	}
```

1. 将前端打包出的dist文件夹拷贝至Nginx根目录下
2. 配置完成后，返回sbin目录下，启动 + 重启 Nginx （Linux系统无需担心重复启动问题）

## 05 修改HOSTS文件

```
#图片上传域名
192.168.126.129  image.jt.com
#后端服务器
192.168.126.129  manage.jt.com
#前端服务器
192.168.126.129  www.jt.com
```

# 4 Bug调试

## sql语句字母大小写问题

说明: 由于Linux系统中严格区分大小写，所以需要将表名全部小写

​                  ![img](https://docimg10.docs.qq.com/image/okNFYsyq7WtNpLCQC85Enw.png?w=1223&h=607)         

## 修改图片上传的地址

​                  ![img](https://docimg3.docs.qq.com/image/flj0sr-hCmKIXlhDLmlB6w.png?w=1142&h=376)    







## yum商店镜像异常

在Centos 8上需要使用yum命令,但执行yum命令时报错 No URLs in mirrorlist。经查阅资料后发现
从2022年1月31日起，CentOS开发团队将会移除官方镜像源上关于CentOS 8所有的包，届时如果在CentOS 8上再次使用yum命令安装包则会报以下错误：下载元数据失败：Cannot prepare internal mirrorlist: No URLs in mirrorlist

解决思路
如果还需要继续使用Centos 8,则需更换下载源

```
解决方法
直接运行命令——更换资源
sudo sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*


缺少libpng12，更换资源后，安装
yum -y install libpng12

libnsl
```



## 运行.sh文件无权限

```
> chmod 777 xxx.sh
```

