# Ubuntu Server 16.04.1 LTS 64位使用vsftpd搭建ftp服务器

### 查看是否安装ftp服务器

```bash
vsftpd -v 
```

### 安装vsftpd

```bash
sudo apt-get install vsftpd
```
如果安装失败或者配置出现问题，可以卸载ftp服务器
```bash
sudo apt-get purge vsftpd 
```

### 创建共享文件夹

```bash
sudo mkdir /home/ftp
sudo chmod 777 /home/ftp
```

### 创建用户

```bash
#添加用户
sudo useradd -d /home/ftp -s /bin/bash ftpuser
#设置用户密码 
sudo passwd ftpuser
#设置ftp目录用户权限
sudo chown ftpuser:ftpuser /home/ftp
```

### 修改/etc/vsftpd.conf

```bash
sudo vim /etc/vsftpd.conf
```
```bash
# 阻止 vsftpd 在独立模式下运行
listen=NO                 
# vsftpd 将监听 ipv6 而不是 IPv4
listen_ipv6=YES           
# 关闭匿名登录
anonymous_enable=NO       
# 允许本地用户登录
local_enable=YES          
# 启用可以修改文件的 FTP 命令
write_enable=YES          
# 本地用户新增档案时的umask 值
local_umask=022           
# 当用户第一次进入新目录时显示提示消息
dirmessage_enable=YES     
# 显示在您的本地时区的时间目录列表
use_localtime=YES         
# 一个存有详细的上传和下载信息的日志文件
xferlog_enable=YES        
# 在服务器上针对 PORT 类型的连接使用端口 20（FTP 数据）
connect_from_port_20=YES  
# 不单独建立ftp用户，直接使用Ubuntu桌面用户就可以登陆
chroot_local_user=YES
chroot_list_enable=NO

#添加vsftpd登录用户对目录树的权限

#新建文件/etc/vsftpd.chroot_list，设置可列出、切换目录的用户：
chroot_list_file=/etc/vsftpd.chroot_list   
# 使用uft8文件系统
utf8_filesystem=YES

# 能够登录的用户名单
userlist_enable=YES
userlist_deny=NO
userlist_file=/etc/vsftpd.user_list

# 锁定一个共享目录
local_root=/home/ftp
# 给共享目录添加写权限
allow_writeable_chroot=YES
```

### 创建 vsftpd.chroot_list 文件添加vsftpd登录用户

```bash
#新建文件/etc/vsftpd.user_list，用于存放允许访问ftp的用户：
sudo touch /etc/vsftpd.user_list
sudo vim /etc/vsftpd.user_list
#在/etc/vsftpd.user_list中添加允许登录ftp 的用户，一行一个
#ftpuser

#添加vsftpd登录用户对目录树的权限
#新建文件/etc/vsftpd.chroot_list，设置可列出、切换目录的用户：
sudo touch /etc/vsftpd.chroot_list
sudo vim /etc/vsftpd.chroot_list
#在/etc/vsftpd.chroot_list 设置可列出、切换目录的用户
#ftpuser
```

### 重启 ftp 服务器

```bash
sudo service vsftpd restart
```

### 浏览器登录ftp服务器

ftp://127.0.0.1


### 参考目录
<https://www.cnblogs.com/wayneliu007/p/10322471.html>
<https://blog.csdn.net/soslinken/article/details/79304076>
<https://blog.csdn.net/qq_28959531/article/details/78631417>