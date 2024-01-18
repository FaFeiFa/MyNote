在Windows系统下，如果你忘记了MySQL的密码，可以通过以下步骤重置密码：

**1.停止MySQL服务：** 打开命令提示符（Command Prompt）或者PowerShell，并使用管理员权限运行。执行以下命令停止MySQL服务：

```bash
net stop mysql
```

**2.以跳过授权表的方式启动MySQL：** 在命令提示符或PowerShell中执行以下命令，以跳过授权表启动MySQL服务：

```mysql
mysqld --console --skip-grant-tables
```

**错误1：**在执行 `mysqld --skip-grant-tables` 时遇到 `'mysqld' 不是内部或外部命令` 的错误:

切换到MySQL安装目录：找到MySQL安装的目录，然后使用 `cd` 命令切换到该目录。例如：

```
cd C:\Program Files\MySQL\MySQL Server X.X
```

**错误2：**[Server] Failed to set datadir to 'C:\Program Files\MySQL\MySQL Server 8.0\data\' (OS errno: 2 - No such file or directory)

```mysql
mysqld --initialize
```

**错误3：**[ERROR] [MY-010131] [Server] TCP/IP, --shared-memory, or --named-pipe should be configured on NT OS

用这个方式启动MySQL

```mysql
mysqld --console --skip-grant-tables --shared-memory
```

**3.以管理员身份打开新的命令提示符或PowerShell：** 打开另一个命令提示符或PowerShell实例，并以管理员身份运行。

```mysql
mysql -u root
```

**4.登录MySQL服务器：** 连接到MySQL服务器，不需要密码：

**5.更新密码：** 在MySQL命令行中执行以下SQL语句，替换`new_password`为你想要设置的新密码：

（这里需要注意**mysql8.0以上版本修改密码方式与以前版本不同，密码格式要求至少包含了数字、字母及特殊字符三种**）

 **使用以前的方式(update user set password=password('123456') where user='root')修改密码时会修改失败，会提示:Found invalid password for user: 'root@localhost'; Ignoring user**

```mysql
use mysql;
update user set authentication_string='' where user='root';// 如果这个字段有值，先置为空，之前的版本密码字段是password
flush privileges;// 刷新权限表
select user,host from user;// 查看用户及host,方便后续修改
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'root@123';// 修改root 密码
exit;//退出mysql
```

**6.退出MySQL命令行：** 执行以下命令退出MySQL命令行：

```bash
quit
```

**7.停止MySQL服务：** 回到之前的命令提示符或PowerShell实例，执行以下命令停止MySQL服务：

```bash
net stop mysql
```

**8.重新启动MySQL服务：** 以正常方式启动MySQL服务：

```mysql
net start mysql
```

现在，你的MySQL密码已经被重置为新设置的密码。请确保新密码是强密码，并妥善保存。

















