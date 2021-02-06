---
title: 阿里云Centos7.4服务器配置jdk、tomcat、mysql教程
date: 2019-04-02 00:54:31
tags: 阿里云Centos
---
## 一.配置jdk1.8

 这个没啥好说的，直接输入指令安装 
```
yum -y install java-1.8.0-openjdk.x86_64
```

安装成功后会有Complete提示，并且可通过java -version查看当前版本

![e3cb0c4c0f022237be7912d8784284e0.png](aliCloud_1/2.png)

 
## 二.搭建tomcat8

 

 用xftp工具与服务器连接后，将下载好的tomcat压缩包丢到/usr/local/目录下，再使用Xshell工具，输入命令
 
```
tar -zxvf apache-tomcat-8.5.39.tar.gz 
```
 解压到当前目录，并将文件名改为tomcat8，之后就能将压缩包删了
![91eeef02156e8b32458011da02ecce6f.png](aliCloud_1/3.png)

 阿里云服务器设置开放8080端口

![7e55bd3204e3a5afbf44ef2e497c2871.png](aliCloud_1/4.png)

![11b852296833403174608a717949382b.png](aliCloud_1/5.png)

 防火墙开放8080端口
```
开放端口：firewall-cmd --zone-public --add-port=8080/tcp --permanent
重新加载：firewall-cmd --reload
```
![afca678e829636d9add04bcc2502eeec.png](aliCloud_1/6.png)

 启动tomcat测试
 
![279ad7b102351a7d57d99ec3ea6bcbc1.png](aliCloud_1/7.png)

![8180f63d54178a788d7519463d16a639.png](aliCloud_1/8.png)

 提高阿里云服务器部署的tomcat外界访问速度，以下为原文地址

 [https://blog.csdn.net/qq_40386113/article/details/84837881](https://blog.csdn.net/qq_40386113/article/details/84837881)

![773c99f1fec5e1b34e741958fb90d966.png](aliCloud_1/9.png)
 
## 三.mysql安装配置

 执行命令, 安装mysql
```
 yum -y install mysql-community-server
```

![c3e681f10fae02b14d6457061850f187.png](aliCloud_1/10.png)

 检查是否安装成功

![f6464c8512f83e80f025952db182eccc.png](aliCloud_1/11.png)


 设置开机自动启动mysql，使用以下两个命令
```
 systemctl enable mysqld
 systemctl list-unit-files | grep mysqld
```
 
 查看是否设置成功

![439b32303bbf837d43f4cd68a5c478ac.png](aliCloud_1/12.png)

启动mysql服务，并查看mysql服务是否已启动
```
systemctl start mysqld
ps -ef|grep mysql
```
![d16b8773edb7b2777374d9a574088661.png](aliCloud_1/13.png)

 进行mysql的用户名、密码等信息进行配置
 
```
mysql_secure_installation
```
![04a407fe51dd10627c11acd3ba551754.png](aliCloud_1/14.png)

![8a238f144ba030e1ec44ce1d899080fd.png](aliCloud_1/15.png)

 启动mysql
 
```
mysql -uroot -p
```

![c0b70c501c48d48e8aa969f4ca509f49.png](aliCloud_1/16.png)

 防火墙开放3306端口
 
```
firewall-cmd --zone-public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

![6851372672b8e250b33fa79fcf0ec75a.png](aliCloud_1/17.png)



 配置远程登录

```
grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;
```

 （指令意为将所有权限赋予给 root 用户，并允许其进行远程登录）

 红框部分填写的是访问远程访问root用户所使用的密码

![c15ef91aed759f37a7c7305a49209b62.png](aliCloud_1/18.png)

 最后，用其他机子测试连接成功
![e34677cd4d3984dfd506876f7ef97226.png](aliCloud_1/19.png)

 

   
 