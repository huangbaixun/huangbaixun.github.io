### 决战大数据之二：CentOS 7 最新JDK 8安装
[TOC]

#### 修改hostname
```
# hostnamectl set-hostname node1 --static
# reboot now

```
重新登陆后你会发现的提示的头为`root@node1`

下载wget，用户网络资源下载，后面的hadoop安装包都需要使用wget来下载 [^yum_istall]

```shell
yum -y update 
yum install weget
```

下载 最新的JDK包并解压
```shell
# cd /opt/
# wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.tar.gz"

# tar xzf jdk-8u101-linux-x64.tar.gz

```

使用alternative来安装JDK

```
[root@note1 opt]# cd /opt/jdk1.8.0_101/
# alternatives --install /usr/bin/java java /opt/jdk1.8.0_101/bin/java 2
# alternatives --config java

There is 1 program that provides 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           /opt/jdk1.8.0_101/bin/java

Enter to keep the current selection[+], or type selection number: 1
# java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```
目前jdk已经安装成功，我们也推荐你使用alternatives安装jre,javac和jar

```shell
# alternatives --install /usr/bin/jar jar /opt/jdk1.8.0_101/bin/jar 2
# alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_101/bin/javac 2
# alternatives --set jar /opt/jdk1.8.0_101/bin/jar
# alternatives --set javac /opt/jdk1.8.0_101/bin/javac
# java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```


[^yum_istall]: http://idroot.net/tutorials/how-to-install-wget-on-centos/

参考：http://tecadmin.net/install-java-8-on-centos-rhel-and-fedora/#



