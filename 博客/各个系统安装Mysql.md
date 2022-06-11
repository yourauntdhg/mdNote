# Centos7.6安装Mysql5.7

## 第一步：安装wget（有则跳过）

> yum -y install wget

知识小讲堂：Linux wget是一个下载文件的工具，它用在命令行下。对于Linux用户是必不可少的工具，尤其对于网络管理员，经常要下载一些软件或从远程服务器恢复备份到本地服务器。如果我们使用虚拟主机，处理这样的事务我们只能先从远程服务器下载到我们电脑磁盘，然后再用ftp工具上传到服务器。这样既浪费时间又浪费精力，那不没办法的事。而到了Linux VPS，它则可以直接下载到服务器而不用经过上传这一步。wget工具体积小但功能完善，它支持断点下载功能，同时支持FTP和HTTP下载方式，支持代理服务器和设置起来方便简单。

## 第二步：下载Mysql安装包

> wget http://dev.mysql.com/get/mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar

## 第三步：centos7自带数据库删除

检查是否有自带数据库maridb：`rpm -qa | grep mari`

卸载命令：`rpm -e --nodeps maridb-libs`

## 第四步：解压安装包

命令：`tar -vxf tar压缩包名称`

## 第五步：安装MySQL

使用`rpm -ivh rpm名称`依次安装mysql-community-common\mysql-community-libs\mysql-community-client\mysql-community-server。命令没有打乱执行过，不知道不按这个顺序会怎么样。

## 第六步：验证并启动

`mysql -V`命令查看是否安装成功。使用`systemctl start mysqld.service`命令启动mysql。

## 第七步：获取初始密码并登录

命令`grep "password" /var/log/mysqld.log`查看初始密码。使用命令`mysql -u root -p`登录mysql，并输入初始密码登录。

## 第八步：修改密码

在MySQL面板中执行：`set password for 'root'@'localhost' =password('修改后的密码');`.

如果密码简单需要修改设置密码时的安全等级为LOW。执行：`set global validate_password_policy=LOW;`

> 关于 mysql 密码策略相关参数；
> 1）、validate_password_length 固定密码的总长度；
> 2）、validate_password_dictionary_file 指定密码验证的文件路径；
> 3）、validate_password_mixed_case_count 整个密码中至少要包含大/小写字母的总个数；
> 4）、validate_password_number_count 整个密码中至少要包含阿拉伯数字的个数；
> 5）、validate_password_policy 指定密码的强度验证等级，默认为 MEDIUM；
> 关于 validate_password_policy 的取值：
> 0/LOW：只验证长度；
> 1/MEDIUM：验证长度、数字、大小写、特殊字符；
> 2/STRONG：验证长度、数字、大小写、特殊字符、字典文件；
> 6）、validate_password_special_char_count 整个密码中至少要包含特殊字符的个数；