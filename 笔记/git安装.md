# 安装

一直点击下一步

# 使用

1.设置姓名和邮箱

git config --global user.name

git config --global user.email

2.配置文件

C:\Users\Hua\.gitconfig

3.设置ssh免密登陆

删除文件夹`C:\Users\用户\.ssh`

在`C:\Users\用户`下打开git输入

```git
ssh-keygen -t rsa -C 邮箱
```

将`C:\Users\用户\.ssh`下的`id_rsa.pub`文件复制并添加到github账户下
