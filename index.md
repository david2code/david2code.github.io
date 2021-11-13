## git使用过程中的各种错误

git使用过程中出现的各种错误及相应解决办法汇总。


### clone github仓库时出错

#### 现象如下：
```c
[root@davidpc ~]# git clone https://github.com/david2code/david2code.git
Cloning into 'david2code'...
fatal: unable to access 'https://github.com/david2code/david2code.git/': Encountered end of file
```
#### 解决办法如下：

将https修改为git，重新执行即可。
```c
git clone git://github.com/david2code/david2code.git
```

