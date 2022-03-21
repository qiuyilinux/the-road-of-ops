

# 一、 基础命令

## 1 提交修改到缓存区

```shell
git add *
```



## 2 查看暂存区

```shell
git status
```



## 3 提交暂存区内容到库

```shell
git commit -m "some"
```



## 4 查看本地分支

```shell
git branch
```



## 5 查看远程分支

```shell
git remote
```



# 二、 场景

## 1 本地分支重命名

```shell
git branch -m oldName newName
```



## 2 远程分支重命名

### a. 重命名远程分支对应的本地分支

```shell
git branch -m oldName newName
```



### b. 删除远程分支

```shell
git push --delete origin oldName
```



### c. 上传新命名的本地分支

```shell
git push origin newName
```



### d. 把修改后的本地分支与远程分支关联

```shell
git branch --set-upstream-to origin/newName
```



## 3 添加远程分支

```shell
git remote add origin https://git.com/gongbosheng/resumes-api.git
```



## 4 取消 ssl 证书的检测

> 当自签证书的时候需要用到

```shell
git config --global http.sslVerify false
```



## 5 HTTPS 免密

```shell
git config --global credential.helper store

# 输入一次之后就不需要输入了 也可以设置成 cache 然后设置 cache 时间（密码保存时间）
```

