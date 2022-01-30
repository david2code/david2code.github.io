# ubuntu
## sudo速度慢的解决办法
hostname得到计算机名称
在/etc/hosts中添加一行
```shell
127.0.0.1   计算机名   计算机名.localdomain
```
## ubuntu usb不识别
- 现象
  lsusb
  lsblk
  无新设备出现
- 解决
  虚拟机关机，在设置中添加usb空的筛选器
  
# 系统时区不正确的解决办法
```shell
timedatectl set-timezone "Asia/Shanghai"
```
# firewall-cmd
## 启停
```shell
systemctl status firewall-cmd
systemctl start firewall-cmd
systemctl stop firewall-cmd
systemctl restart firewall-cmd
```
## 查看当前配置
```shell
firewall-cmd --zone=wan --list-all
```
## 放行端口
### 命令行
```shell
firewall-cmd --permanent --zone=wan --add-port=3306/tcp
```
###  直接修改配置文件
```shell
vi /etc/firewalld/zones/wan.xml
<port protocol="tcp" port="3306"/>
firewall-cmd --reload
```
# samba
## Linux主机修改了用户密码，windows不提示密码错误，无法登陆。
- 控制面板-用户账户-管理你的凭据-windows凭据-编辑samba的账户密码
# mysql
## Failed to find valid data directory
- 一般是因为存放数据的目录被毁坏或者不存在：比如安装mysql时，磁盘已满，结果数据目录创建不成功
- 打开 /etc/my.conf，找到datadir
- 清空 datadir 目录下的所有内容
- 执行 /usr/bin/mysqld_safe --initialize 将会生成带随机密码的root账户
- root 密码，可以在datadir目录下的d.err文件中找到
- 登陆后要先修改默认密码
```shell
alter user user() identified by "XXXXXX";
```
# windows子系统ubuntu安装软件出错
> Could not read response to hello message from hook [ ! -f /usr/bin/snap ] ||
- 解决办法
```shell
sudo rm -rf /etc/apt/apt.conf.d/20snapd.conf
```

# 配置rsyslog日志服务器
## 配置server
- apt install rsyslog rsyslog-relp
- apt install rsyslog-mysql
  安装过程中要配置mysql账户，按默认即可
  ==注意==
  
  1. mysql的默认sock文件在/tmp/mysql.sock，此安装程序找不到，因此要修改mysql的默认sock位置
  2. 记住创建的mysql账户和数据库名，后面要使用
- 开启server模式
  ```shell
  vi /etc/rsyslog/rsyslog.conf
  
  module(load="imrelp")
  input(type="imrelp" port="514")
  
  systemctl restart rsyslog
  ```
## 配置client
- apt install rsyslog rsyslog-relp

```shell
vi /etc/rsyslog.d/rsyslog.conf
module(load="omrelp")

vi /etc/rsyslog.d/50-default.conf
*.*     :omrelp:192.168.200.63:514
systemctl restart rsyslog
```
## 安装 loganalyzer
- 安装lnmp1.8-full.tar.gz
  - mysql 8.0.25
  - php 8.0.8
- 配置mysql
```shell
vi /etc/my.conf

mkdir /var/run/mysqld
socket      = /var/run/mysqld/mysqld.sock
systemctl restart mysql
mysql -uroot -p
create database loganalyzer;
```
- 配置php
```shell
vi /usr/local/php/etc/php.ini

找到 disable_functions，去掉syslog,openlog
打开 extension=mysqli
mysqli.default_socket = /var/run/mysqld/mysqld.sock

systemctl restart php-fpm
```
- 下载 loganalyzer-4.1.12.tar.gz
  - 将src拷贝到/home/wwwroot 目录
  - 配置nginx
  ```shell
  vi /usr/local/nginx/conf/nginx.conf
  
  root  /home/wwwroot/loganalyzer;
  
  systemctl restart nginx
  ```
  - 修改loganalyzer代码
  ```shell
  vi /home/wwwroot/loganalyzer/include/functions_common.php
  
  function RemoveMagicQuotes()
  {
      //if (get_magic_quotes_gpc()) {
      if (true) {
          $process = array(&$_GET, &$_POST, &$_COOKIE, &$_REQUEST);
          //while (forlist($key, $val) = each($process)) {
          foreach ($process as $key => $val)
              foreach ($val as $k => $v) {
                  unset($process[$key][$k]);
                  if (is_array($v)) {
                      $process[$key][stripslashes($k)] = $v;
                      $process[] = &$process[$key][stripslashes($k)];
                  } else {
                      $process[$key][stripslashes($k)] = stripslashes($v);
                  }
              }
          //}
          unset($process);
      }
  }
  ```
- 安装loganalyzer
打开浏览器
http://ip:port/


