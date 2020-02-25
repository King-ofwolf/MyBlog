# ubuntu 16.04 server 开启ssh

### 1. 安装
```bash
sudo apt-get install -y openssh-server
```
检查是否安装成功(出现sshd)
```bash
ps -e | grep ssh
```
### 2. 开启ssh服务
```bash
sudo service ssh start
```
### 3. 修改ssh可以使用root登陆
```bash
sudo vim /etc/ssh/sshd_config
```
找到PermitRootLogin，改成yes，保存并重启ssh
```bash
sudo service ssh restart
```


