# HadoopGuideline
my guideline (in Chinese) to install Hadoop with VMPlayer

我的host机器是windows 10，安装了对个人免费的VMWare Player，准备使用4个虚拟机，都安装Ubuntu Server 16.04.1 LTS。其中master作为namenode，slave1, slave2, slave3作为datanode。

## 安装Ubuntu
安装VMPlayer，下载Ubuntu Server不必赘述，默认安装即可。安装完之后sudo apt update && sudo apt upgrade -yy更新一下。

## 基本配置
创建Hadoop用户组，安装SSH，安装OpenJava, 下载解压Hadoop，配置SSH免密码
