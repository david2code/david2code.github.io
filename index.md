# Linux

[https://david2code.github.io/linux]

# windows
# git
## git使用过程中的各种错误

git使用过程中出现的各种错误及相应解决办法汇总。


### clone github仓库时出错

#### 现象如下：
```shell
[root@davidpc ~]# git clone https://github.com/david2code/david2code.git
Cloning into 'david2code'...
fatal: unable to access 'https://github.com/david2code/david2code.git/': Encountered end of file
```
#### 解决办法如下：

将https修改为git，重新执行即可。
```shell
git clone git://github.com/david2code/david2code.git
```
### github push代码不成功
在setting找到 SSH keys and GPG keys，点击New SSH key
将本地环境中 .ssh/id_rsa.pub 的内容复制粘贴进去。如果没有，则使用ssh-keygen生成。
### git clone http 出错
server certificate verification failed. CAfile: none CRLfile: none
#### 解决办法如下：
vi ~/.gitconfig
设置不校验http/https
```shell
[user]
    email = david2code@github.com
    name = david
[http]
    sslverify = false
[https]
    sslverify = false
```

### git忽略文件权限或者所有者改变
```shell
git config --global core.fileMode false
```
或者设置 .git/config
```shell
core  filemode = false
```

### 远程新建了一个空仓库，如何将本地的代码仓库推送到远程仓库？
- 在本地仓库查看当前远程库链接
```shell
git remote -v
```
-  删除当前的远程库链接
```shell
git remote rm origin
```
- 添加要合并的远程仓库连接
```shell
git remote add origin https://192.168.250.1/gitea/default/david2code.git/
```
- 初始化本地仓库
```shell
git init
```
- 推送到远程
```shell
git push origin master
```
- 如果出现如下错误
```shell
fatal: refusing to merge unrelated histories
```
- 使用如下命令
```shell
git pull origin master --allow-unrelated-histories
```

# git 撤回commit add push
## 撤回commit，不删除改动的代码
```shell
git reset --soft HEAD^
```
## 撤回commit，删除改动的代码
```shell
git reset --hard HEAD^
```
## 撤回add
```shell
git reset HEAD
```
## 撤回push
```shell
git reset --hard commit-id
git push origin branch --force
```
## 撤回git rm
```shell
git reset
git checkout
```
## git放弃当前所有未add的修改
git reset --hard
## git 删除当前不在版本控制的文件，如编译中间文件等
git clean -f
其中-f表示不删除 .gitignore中指定的文件

## 删除分支
- 删除本地分支
```shell
git branch -d branch_name
```
- 删除运程分支
```shell
git push origin -d branch_name
```
## git 显示分支历史
```shell
git log --graph --all
```
## git获取commit-id
```shell
git rev-parse HEAD
git rev-parse --short HEAD
```

## git status 中文乱码
```shell
git config --global core.quotepath false
```
## 对git的历史提交生成patch
```shell
git format-patch -2 git-ver1 git-ver2
```
## 应用patch
```shell
git apply --check 0001.patch
git apply 0001.patch
```

