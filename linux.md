# ubuntu sudo速度慢的解决办法
hostname得到计算机名称
在/etc/hosts中添加一行
```c
127.0.0.1   计算机名   计算机名.localdomain
```
# 系统时区不正确的解决办法
```
timedatectl set-timezone "Asia/Shanghai"
```
