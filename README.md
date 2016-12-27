# Hadoop Guideline
##### by Erich Chen

my guideline (in Chinese) to install Hadoop with VMPlayer

我的host机器是windows 10，安装了对个人免费的VMWare Player，准备使用4个虚拟机，都安装Ubuntu Server 16.04.1 LTS。其中master作为namenode，slave1, slave2, slave3作为datanode。

hostname    | IP address     | Role
------------|----------------|----------
master      | 192.168.88.100 | namenode
slave1      | 192.168.88.101 | datanode
slave2      | 192.168.88.102 | datanode
slave3      | 192.168.88.103 | datanode


## 安装Ubuntu
安装VMPlayer，下载ubuntu-16.04.1-server-amd64.iso不必赘述，默认的快速无人值守安装即可。
安装过程中要求的机器名、用户名等信息可以随意设置，后面会再增加hadoop组和hduser。网络方式确认一下是NAT，默认就是NAT。
安装完之后用自己的用户名登录进去，更新一下系统。  
```
sudo apt update && sudo apt upgrade -yy
```

## 基本配置
```bash
sudo apt install vim -y   # 安装vim，可选；不使用vim的话，系统自带vi和nano。

###修改IP地址
ifconfig        #检查网卡名称（ens33），当前IP地址（192.168.88.*)
route -n        #检查默认网关(192.168.88.2)

sudo vim /etc/network/interfaces
# 添加下面几行，设置固定IP地址
    auto ens33
    iface ens33 inet static
    address 192.168.88.8x
    gateway 192.168.88.2
    netmask 255.255.255.0
    network 192.168.88.0
    broadcast 192.168.88.255
    dns-nameservers 192.168.88.2
    
sudo vim /etc/hostname      #默认是ubuntu，改成master

sudo vim /etc/hosts
#添加下面几行
    192.168.88.100  master
    192.168.88.101  slave1
    192.168.88.102  slave2
    192.168.88.103  slave3    

sudo reboot #重启后检查配置是否已生效
```

## 下载安装JDK和Hadoop
这里大量参考了Digital Ocean的一个教程。详细解释请参阅：
https://www.digitalocean.com/community/tutorials/how-to-install-hadoop-in-stand-alone-mode-on-ubuntu-16-04

```
# 更新一下sourcelist
sudo apt update

# 一起安装SSH，hadoop要通过SSH管理各个节点，后面再详细配置
sudo apt install openssh-server -y

# 安装OpenJava
sudo apt install default-jdk -y
java -version  #检查java版本，当前是openjdk-1.8.0_111

# 下载并解压Hadoop
# 先去找到最快的下载地址：http://hadoop.apache.org/releases.html
# 当前版本是2.7.3，推荐给我的下载地址是：
wget http://apache.mirrors.tds.net/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz
tar -xzvf hadoop-2.7.3.tar.gz           #解压
sudo mv hadoop-2.7.3 /usr/local/hadoop  #移动到这个位置方面多用户访问   

# 为Hadoop配置JAVA环境变量，只需要改下面一处即可
sudo vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
    export JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")
# 这里用了个技巧读取default java的位置，详见Digital Ocean的教程。

#创建hadoop用户组，并添加用户hduser
sudo addgroup hadoop
sudo adduser --ingroup hadoop hduser
# NOTE：hduser并没有sudo权限，增强安全性  

# 把hadoop文件夹的owner改为hduser用户和hadoop组
sudo chown -R hduser:hadoop /usr/local/hadoop

# 关机
sudo poweroff
```

## 复制虚拟机

将虚拟机关机后，复制四份。只需要复制.vmx和.vmdk文件。
双击.vmx打开虚拟机，遇到提示时选择"I copied it"。
**注意一次只打开一个虚拟机，避免IP地址冲突**
```bash
1. 修改虚拟机名字，分别改名为master, slave1, slave2, slave3（可选，只为方便识别）；
2. 必须修改IP地址和主机名   
sudo vim /etc/hostname      #分别修改为master, slave1, slave2, slave3
sudo vim /etc/network/interfaces    # address行分别修改为192.168.88.100/101/102/103

sudo reboot     #修改完之后，重启生效
```
**#####################注意##############################**   
**在此之前都是用主用户登录。在此之后都是用hduser用户登录。**
**#######################################################**   

## 测试hadoop
```bash
/usr/local/hadoop/bin/hadoop    #显示hadoop usage帮助
```

## 配置SSH免密码
**再次提醒hduser登录，并保持四台机器都开机**  
在四台机器上用hduser用户登录后，分别依次执行下面的语句：
```bash
ssh-keygen -t rsa -P ''   #注意最后是两个单引号
ssh-copy-id -i /home/hduser/.ssh/id_rsa.pub master
ssh-copy-id -i /home/hduser/.ssh/id_rsa.pub slave1
ssh-copy-id -i /home/hduser/.ssh/id_rsa.pub slave2
ssh-copy-id -i /home/hduser/.ssh/id_rsa.pub slave3

# 测试，都应该直接连接上，不需要输入密码
ssh master
ssh slave1
ssh slave2
ssh slave3
```

## Hadoop Cluster配置
```bash
到这里基本上可以自由发挥了。
官方手册应该是最好的manul了。
http://hadoop.apache.org/docs/r2.7.3/hadoop-project-dist/hadoop-common/ClusterSetup.html
```
