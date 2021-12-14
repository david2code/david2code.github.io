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

### git clone http 出错
server certificate verification failed. CAfile: none CRLfile: none
#### 解决办法如下：
vi ~/.gitconfig
设置不校验http/https
```c
[user]
    email = lss-1120@3onedata.com
    name = Lu ShanShan
[http]
    sslverify = false
[https]
    sslverify = false
```

### git忽略文件权限或者所有者改变
git config --global core.fileMode false
或者设置 .git/config
```c
core  filemode = false
```

### 远程新建了一个空仓库，如何将本地的代码仓库推送到远程仓库？
#### 在本地仓库查看当前远程库链接
git remote -v
#### 删除当前的远程库链接
git remote rm origin
#### 添加要合并的远程仓库连接
git remote add origin https://192.168.250.1/gitea/default/ls-gzxt-zhdg.git/
#### 初始化本地仓库
git init 
#### 推送到远程
git push origin master
#### 如果出现如下错误
fatal: refusing to merge unrelated histories
#### 使用如下命令
git pull origin master --allow-unrelated-histories


