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

# 修改shell命令行不显示绝对路径
将.bashrc 中的w改成大写即可
```shell
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\W\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\W\$ '
fi
```
# fatal error: systemd/sd-bus.h: No such file or directory
```shell
apt install libsystemd-dev
```

# Linux主机之间免密登陆
假设A想免密登陆B
- 在A主机ssh-keygen生成id_rsa.pub
- 在B的.ssh目录新建（如果没有)authorized_keys文件，将A的id_rsa.pub粘贴进去

# postgresql
- 安装
```shell
apt install postgresql
```
- 设置默认帐号postgres密码
```shell
sudo su postgres
psql
alter user postgres with password '123456'
```
- 创建数据库并查看和进入
```shell
create database my_db
\l
\c my_db
\q
```
- 使用账号登陆
```shell
psql -U postgres -h 127.0.0.1 -p 5432 my_db
```
- 查询表格和自定义表格和表结构
```shell
select * from pg_tables;
select * from pg_tables where schemaname = 'public';
\d my_table
```
- 导出建表语句
```shell
pg_dump -h 127.0.0.1 -p 5432 -U postgres -d my_db -t pg_opclass -O -s > aa.sql
```
- 备份数据表
```shell
pg_dump -h localhost -p 5432 -U postgres -d my_db -t table* -O > aa.sql
```
- 恢复数据
```shell
psql -h localhost -p 5432 -U postgres my_db < aa.sql
```
- 查看当前所有角色
```shell
select rolname from pg_roles;
```
- 查看当前所有帐号
```shell
select * from pg_user;
```
- 修改用户密码
```shell
alter user postgres password '123456';
```
- postgresql 查看正在执行的命令
SELECT
  procpid,
  start,
  now() - start AS lap,
  current_query
FROM
  (SELECT
    backendid,
    pg_stat_get_backend_pid(S.backendid) AS procpid,
    pg_stat_get_backend_activity_start(S.backendid) AS start,
    pg_stat_get_backend_activity(S.backendid) AS current_query
  FROM
    (SELECT pg_stat_get_backend_idset() AS backendid) AS S
  ) AS S
WHERE
  current_query <> '<IDLE>'
ORDER BY
  lap DESC;

- kill 进程

SELECT pg_cancel_backend(进程id);


- 查处空闲进程
SELECT
 pid,
 datname AS db,
 query_start AS start,
 now() - query_start AS lap,
 query
FROM pg_stat_activity
WHERE state <> 'idle' and query not like '%pg_stat_activity%'
 and (now() - query_start) > interval '10 seconds';

- 复制表结构，但不复制索引和约束
create table test_bak (like test);
## 表结构复制，包换索引和约束
create table test_bak (like test including all);
## 表结构和数据复制
create table test_bak as (select * from test where column_name = something);

- 重命名表
alter table test rename to test_bak;

- vacuumdb
- 有一个表数据非常多，结果查询时非常慢。于是就把数据几乎都删除了，只剩下几百条。结果查询还是很慢。检查发现，虽然数据删除了，但表所占用的存储没有变化。原来postsql并不会立即清理存储，需要使用如下命令清理，清理后查询速度就很快了
vaccumdb -U postgres -h localhost -p 5432 -d database_name -t table_name

- 使用truncate清空表，可以立即回收存储空间，可以不使用vacuumdb
truncate table table_name;
 
# linux合并两个文件夹
```shell
cp -rap dir1 dir2
```

# git 与 xargs 与 cp 命令搭配使用
```shell
git diff --name-only commit-id  |xargs -t -i cp {} ../tmp/
```
git diff --name-only 是将当前版本与后面指定版本进行比较，显示差异的文件名
xargs -t  表示将后面要执行的命令一行一行打印出来 可选
      -i 表示将xargs的每项名称，一行一行的赋给{}

# qemu模拟ARM系统
https://zhuanlan.zhihu.com/p/340362172
## 安装
sudo apt install qemu qemu-utils

## 编译内核镜像
### 下载内核源码
### 配置
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_defconfig
### 内核裁剪
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
### 编译
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8
### 得到zImage
arch/arm/boot/zImage
arch/arm/boot/dts/vexpress-v2p-ca9.dtb

## 编译u-boot
## 下载u-boot源码
git clone git://git.denx.de/u-boot.git
## 编译
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- vexpress_ca9x4_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8
## 仿真
qemu-system-arm -M vexpress-a9 -m 256 -kernel u-boot -nographic

## busybox
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- install

## 制作rootfs
- mkdir -p rootfs/{dev,etc/init.d,lib}
- cp busybox-1.35.0/_install/* rootfs/ -r
- cp -a busybox-1.35.0/examples/bootfloppy/etc/* rootfs/etc/
- sudo cp -P /usr/arm-linux-gnueabihf/lib/* rootfs/lib/
- qemu-img create -f raw disk.img 512M
- mkfs -t ext4 ./disk.img
- 将rootfs中的所有文件复制到disk.img
```shell
mkdir tmpfs
sudo mount -o loop ./disk.img tmpfs/
sudo cp -r rootfs/* tmpfs/
sudo umount tmpfs
```

## 启动虚拟机
qemu-system-arm \
        -M vexpress-a9 \
        -m 512M \
        -kernel linux-5.19.3/arch/arm/boot/zImage \
        -dtb linux-5.19.3/arch/arm/boot/dts/vexpress-v2p-ca9.dtb  \
        -nographic \
        -append "root=/dev/mmcblk0 rw console=ttyAMA0" \
        -sd disk.img

## 关闭虚拟机
killall qemu-system-arm

## can't open /dev/tty2: No such file or directory
sudo mknod dev/null c 1 3
sudo ln -s null tty4
sudo ln -s null tty3
sudo ln -s null tty2

## 编写应用程序在qemu系统中运行
mkdir app
vi app/main.c
```c
#include<stdio.h>
#include<unistd.h>

int main()
{
        while(1) {
                printf("run in qemu!\n");
                sleep(1);
        }
}
```
arm-linux-gnueabihf-gcc -o main main.c
qemu-arm -L /usr/arm-linux-gnueabihf/ main

sudo mount -o loop disk.img tmpfs/
sudo cp app/main tmpfs/root/
sudo umout tmpfs
重新运行虚拟机

## 建立虚拟机与宿主机的网络通信  未实现
- sudo apt-get install uml-utilities
- sudo tunctl -u root -t tap30
- ifconfig tap30 192.168.111.1 promisc up
- 启动
  qemu-system-arm \
        -M vexpress-a9 \
        -m 512M \
        -kernel linux-5.19.3/arch/arm/boot/zImage \
        -dtb linux-5.19.3/arch/arm/boot/dts/vexpress-v2p-ca9.dtb  \
        -nographic \
        -append "root=/dev/mmcblk0 rw console=ttyAMA0" \
        -sd disk.img \
        -net nic \
        -net tap,ifname=tap30,script=no,downscript=no

## uboot 通过tftp加载zImage
- vi include/configs/vexpress_common.h
```shell
#define CONFIG_IPADDR 192.168.13.112
#define CONFIG_NETMASK 255.255.255.0
#define CONFIG_SERVERIP 192.168.13.1
#define CONFIG_BOOTFILE "uImage"
#define CONFIG_BOOTCOMMAND "tftp 0x60003000 uImage;setenv bootargs'root=/dev/mmcblk0 console=ttyAMA0';bootm 0x60003000"
```
- 编译u-boot
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8
- 编译kernel
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LOADADDR=0x60003000 -j4
- 启动u-boot
qemu-system-arm -M vexpress-a9 -m 256 -kernel u-boot -nographic -net nic,macaddr=00:25:33:00:00:01 -net tap
