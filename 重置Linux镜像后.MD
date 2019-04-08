### 起因
阿里云主机因为一些情况，导致需要重装，以下是过程，记录一下以后也许能用得上。
> 环境：centos7.4

### 1，修改计算器名
hostnamectl set-hostname ttsx  
重启后生效

### 2，创建个人用户
useradd ttsx

### 3，修改密码
passwd ttsx

### 4，赋予sudo权限
vi /etc/sudoers  
在root ALL=(ALL) ALL”在起下面添加”ttsx ALL=(ALL) ALL”  
退出使用wq!

### 5，安装nginx
yum install nginx  
启动  
systemctl start nginx   
开机启动  
systemctl enable nginx  
设置用户  
/etc/nginx/nginx.conf中设置user为有权访问项目文件夹的用户。  

### 6 安装node
安装nvm  
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash  
执行source .bashrc  
安装node（最新版）  
nvm install node  

### 7，安装apache
yum install httpd  
启动  
systemctl start httpd  
开机启动  
systemctl enable httpd  
注意：80已经被nginx占用，apache启动会失败，需要设置为别的端口  

### 8，安装php7
安装SCL（Software Collections）软件库：  
yum install centos-release-scl  
安装php及一些依赖  
yum install rh-php72 rh-php72-php rh-php72-php-gd rh-php72-php-mbstring rh-php72-php-intl  
安装mysql模块  
yum install rh-php72-php-mysqlnd  
链接php命令  
ln -s /opt/rh/rh-php72/root/usr/bin/php /usr/bin/php  
使用apache运行php  
Symlink the PHP 7.2 Apache modules into place:  
ln -s /opt/rh/httpd24/root/etc/httpd/conf.d/rh-php72-php.conf /etc/httpd/conf.d/  
ln -s /opt/rh/httpd24/root/etc/httpd/conf.modules.d/15-rh-php72-php.conf /etc/httpd/conf.modules.d/  
ln -s /opt/rh/httpd24/root/etc/httpd/modules/librh-php72-php7.so /etc/httpd/modules/   
参考自  
https://docs.nextcloud.com/server/15/admin_manual/installation/php_72_installation.html#rhel-7-upgrade-to-php-7-2

### 9，使用nginx+php-fpm
安装php-fpm  
yum install rh-php72-php-fpm  
启动  
systemctl start rh-php72-php-fpm  
开机启动  
systemctl enable rh-php72-php-fpm  
设置用户  
进入/目录，使用find . -name “php-fpm.conf”找到conf文件，在user和group设置用户，设置为php项目根目录有权访问和修改的用户。  
nginx配置参照  
https://www.nginx.com/resources/wiki/start/topics/recipes/wordpress/  
https://docs.nextcloud.com/server/15/admin_manual/installation/nginx.html  

### 10，安装mysql
yum install rh-mysql57-mysql-server  
临时环境变量  
scl enable rh-mysql57 bash  
永久环境变量  
vi /etc/profile.d/rh-mysql57.sh  
\# create new  
\#!/bin/bash source  
/opt/rh/rh-mysql57/enable  
export X_SCLS=”`scl enable rh-mysql57 ‘echo $X_SCLS’`”  
设置编码  
vi /etc/opt/rh/rh-mysql57/my.cnf.d/rh-mysql57-mysql-server.cnf  
\# add follows within [mysqld] section  
[mysqld]  
character-set-server=utf8  
启动  
systemctl start rh-mysql57-mysqld  
开机启动  
systemctl enable rh-mysql57-mysqld  
设置密码  
执行命令：mysql  
执行：set password for root@localhost = password(‘123’);   
之后进入时使用 mysql -uroot -p123  
参考  
https://www.server-world.info/en/note?os=CentOS_7&p=mysql57

### 11，安装wordpress
下载  
https://cn.wordpress.org/中文版官网，下载贼慢，慢慢下。  
安装  
解压到某个地方，配置apache的documentRoot或者nginx的root，重启。  
打开网页，按照指引输入mysql账户密码，网站信息即可。  

### 12，安装mongodb
yum install rh-mongodb36  
临时环境变量  
scl enable rh-mongodb36 bash  
永久环境变量   
vi /etc/profile.d/rh-mongodb36.sh  
create new  
!/bin/bash  
source /opt/rh/rh-mongodb36/enable  
export X_SCLS=”`scl enable rh- mongodb36 ‘echo   
启动  
systemctl start rh-mongodb36-mongod  
开机启动  
systemctl enable rh-mongodb36-mongod  
设置bindIp  
默认的配置启动失败时 /etc/opt/rh/rh-mongodb36/mongod.conf中设置bindIp为0.0.0.0。  
设置密码  
命令：mongo  
创建admin账户，可以管理其他数据库， 在未设置auth的时候操作   
use admin  
db.createUser({ user: “admin”, pwd: “123456”, roles: [{ role: “userAdminAnyDatabase”, db: “admin” }] })
创建普通用户，只有test数据的读写权限，在未设置auth时或者admin登陆后可操作
use test  
db.createUser({user:”test”,pwd:”123456″,roles:[“readWrite”]})  
打开权限  
/etc/opt/rh/rh-mongodb36/mongod.conf 末尾处找到security取消注释，在它下面找到authorization取消注释并赋值authorization: enabled。想取消密码限制时注释这两处即可。  
使用账户密码：先选择数据use test，再db.auth(‘test’, ‘123456’)。  

### 13，创建swap分区  
内存太小webpack都能跑崩，加个swap分区就好了。  
1. dd if=/dev/zero of=/swapfile bs=1k count=2048000  
2. 创建SWAP文件  
 mkswap /swapfile  
3. 激活SWAP文件  
 swapon /swapfile  
4. 查看SWAP信息是否正确  
 swapon -s  
5. 添加到fstab文件中让系统引导时自动启动  
 echo “/var/swapfile swap swap defaults 0 0” >> /etc/fstab  
6. 用命令free检查2G交换分区生效  
 free -m  
 grep SwapTotal /proc/meminfo  
7. 释放SWAP文件  
 swapoff /swapfile  
8. 删除SWAP文件  
 rm -fr /swapfile  
来自  
https://www.cnblogs.com/jackluo/p/5920463.html