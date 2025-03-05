# 从本地创建一个分支，并同步到remote

现在本地目录初始化一个git仓库
```git
git init
```
然后，在github上创建代码仓，并复制地址
```git 
git remote add origin <url>
```
推送到remote的main分支
```git
git push -u origin main
```
如果本地是master分支，可以选择改下名字
```git
git branch -M main
git push -u origin master:main
```

# 更换本地url

HTTPS和SSH最大的不同在于，HTTPS是通过帐密登录授权的，SSH是通过密钥
HTTPS适合短期跟踪一个项目，而SSH适合长期维护一个项目

首先保障自己有ssh密钥，生成方法
```git
ssh-keygen -t ed25519 -C "zzuljs@gmail.com"

#复制到粘贴板
cat ~/.ssh/id_ed25519.pub
```
注意这里的ed25519是一种加密方法，也可以用RSA

```git
git remote set-url origin <ssh-url>
```
