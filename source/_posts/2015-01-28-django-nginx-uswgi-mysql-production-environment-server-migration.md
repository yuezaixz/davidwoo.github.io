---
layout: post
title: 'Django、nginx、uswgi、mysql生产环境的服务器迁移'
date: 2015-01-28 08:28
comments: true
categories: 
---
#迁移环境
##source

* (此处修改，不公开服务器)
* ubuntu 12.04

##target

* 此处修改，不公开服务器
* ubuntu 14.04.1

#旧服务器备份
##数据备份
```bash
#先连接source
ssh 此处修改，不公开服务器 -l unbx
#使用mysqldump命令，用unbxprod用户登录，备份unbxprod库到用户根目录backUp文件夹中的unitybox_mysql_dump.txt文件
mkdir ~/backUp
mysqldump -u此处修改，不公开服务器 -p 此处修改，不公开服务器 > ~/backUp/unitybox_mysql_dump.txt
```
##Python运行环境备份
使用pip命令备份，如果source环境为安装pip。
```bash
pip freeze > ~/backUp/requirements.txt
```
##ssl文件备份
```bash
cp -r nginx-conf/ backUp/
```
##脚本&nginx & uwsgi配置文件备份
```bash
mkdir ~/backUp/sript 
mkdir ~/backUp/ng_conf 
mkdir ~/backUp/wsgi_conf 
find ~/ -name *.sh -exec cp {} ~/backUp/sript  \
find ~/ -name *.conf -exec cp {} ~/backUp/ng_conf  \
find ~/ -name *.ini -exec cp {} ~/backUp/wsgi_conf  \
```

##备份文件传送
```bash
scp -r /home/unbx/backUp unbx@此处修改，不公开服务器:/home/unbx/
```
#新环境部署
```bash
#先用root连接新服务器，如果unbx用户已创建，请直接用unbx登录
ssh 此处修改，不公开服务器 -l root
```
##系统用户初始
###用户及用户组创建
```bash
#记住这里要adduser，否则unbuntu将不会帮你创建用户主目录
adduser unbx
passwd unbx
vim /etc/group
#配置unbx 组到www-data ，www-data到unbx
```
###设置sudoer
```bash
chmod u+w /etc/sudoers
vim /etc/sudoers
chmod u-w /etc/sudoers
#这时候用户已经创建完成，切换unbx开始工作
su unbx
cd ~/
```
##数据清理
如果mysql在旧机器上未安装，请跳过该章节
```bash
sudo apt-get autoremove --purge mysql-server-5.0
sudo apt-get remove mysql-server
sudo apt-get autoremove mysql-server
sudo apt-get remove mysql-common 
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
```
##安装环境
```bash
sudo apt-get install git mysql-server python-mysqldb python2.7 python2.7-dev
#安装pip
wget https://bootstrap.pypa.io/get-pip.py 
python get-pip.py
sudo python get-pip.py
#安装django
sudo pip install https://www.djangoproject.com/download/1.7c1/tarball/
#安装pillow的jpg和png依赖
sudo apt-get install libjpeg-dev zlib1g*
sudo aptitude install libfreetype6-dev
#安装OpenSSL的依赖
sudo apt-get install libssl-dev
```
##配置MySql字符集
```bash
sudo vim /etc/mysql/my.cnf
```
在my.cnf中添加相应内容，如下所示

```
……
    [client]
    default-character-set=utf8
……
    [mysql]
    default-character-set=utf8
……
    [mysqld]
    collation-server=utf8_unicode_ci
    init-connect='SET NAMES utf8'
    character-set-server=utf8
    default-storage-engine=InnoDB
……
```

然后重启mysql服务

```bash
sudo service mysql restart
```

##设置git的ssh

```bash
#https://confluence.atlassian.com/pages/viewpage.action?pageId=270827678
ssh-v    #看ssh版本
ls-a~/.ssh #看是否存在.ssh目录，有的话备份后删除
ssh-keygen #生成ssh秘钥
ls-a~/.ssh #查看是否生成
ps-e|grep[s]sh-agent  #查看是否有ssh-agent进程，我的做法是由的话杀了
ssh-agent/bin/bash #启动ssh-agent
ps-e|grep[s]sh-agent #查看是否生成
ssh-add~/.ssh/id_rsa #添加信息到秘钥
ssh-add-l
cat~/.ssh/id_rsa.pub #查看公钥，添加到git服务器端的个人配置中
```

把公钥添加到git服务器端的个人ssh配置后，就可以用git下载工程了。注意：【此处修改，不公开服务器】的秘钥时候我设置了密码，密码请参考[Account and Passwords](https://bitbucket.org/unitybox/pyunitybox/wiki/passwords)

```bash
gitclonegit@此处不公开
```

##安装Python运行环境
前面的步骤中我们已经安装了pip了，现在可以让他排上用场了。先修改requirenment.txt，首先Django刚刚我们已经安装了1.7c1，所以Django那行可以删除。另外，有一些依赖在pip环境上已经变更，所以也需要相应的调整，否则会受影响，这里我根据执行中的环境差异做的调整如下，x为已经不存在的。
```bash
pip install -r requirenment.txt 
```
有些环境不一致，可能会报错，很多依赖找不到或者版本不正确，如果遇到可以进行调整，参考如下：

>Twisted-Core==11.1.0 13.2.0

>Twisted-Names==11.1.0 x

>Twisted-Web==11.1.0 x

>apt-xapian-index==0.44 0.45

>cloud-init==0.6.3 0.7.5

>command-not-found==0.2.44 x

>euca2ools==2.0.0 x

>language-selector==0.1 xx

>launchpadlib==1.9.12  1.6.0

>lazr.restfulclient==0.12.0  0.12.2

>pycurl==7.19.0  安装报错

>python-apt==0.8.3ubuntu7.2   0.9.3.5

>unattended-upgrades==0.1 x

>wadllib==1.3.0 1.3.2

>ufw x 

调整完就可以进行安装了
```bash
pip install -r requirenment.txt 
```

##初始化数据库
```bash
mysql -uroot -p
```
```mysql
---初始化数据库用户和数据库
此处不公开
---初始化数据
此处不公开
source /home/unbx/backUp/unitybox_mysql_dump.txt
```

##配置uwsgi和nginx
```bash
sudo ln -s ~/PyUnitybox/Web/src/adm_nginx.conf /etc/nginx/sites-enabled/
sudo ln -s ~/PyUnitybox/Web/src/api_nginx.conf /etc/nginx/sites-enabled/
sudo ln -s ~/PyUnitybox/Web/src/biz_nginx.conf /etc/nginx/sites-enabled/
sudo mkdir /etc/uwsgi
sudo mkdir /etc/uwsgi/vassals
sudo ln -s ~/PyUnitybox/Web/src/adm_uwsgi.ini /etc/uwsgi/vassals
sudo ln -s ~/PyUnitybox/Web/src/api_uwsgi.ini /etc/uwsgi/vassals
sudo ln -s ~/PyUnitybox/Web/src/biz_uwsgi.ini /etc/uwsgi/vassals
#创建nginx的log目录
mkdir log
```

##考核ssl文件
```bash
cp -r ~/backUp/nginx-conf ~/
```
##修改Django的setting配置文件
然后要根据所部署的服务器server name的不同，修改biz/settings.py和adm/settings.py中的相应部分。
```python
此处不公开
```

##启动uwsgi
```bash
sudo vim /etc/rc.local
#增加内容：/usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data
```
#collectstatic
```bash
cd PyUnitybox/Web/src/
python manage_biz.py collectstatic
python manage_adm.py collectstatic
```

##重启测试是否能自动启动uwsgi
```bash
reboot
```

##创建启动脚本
```bash
vim restart_pyunitybox.sh
sudo kill -9 `cat ~/log/unitybox-biz.pid`
sudo kill -9 `cat ~/log/unitybox-adm.pid`
sudo kill -9 `cat ~/log/unitybox-api.pid`

chmod +x restart_pyunitybox.sh 
./restart_pyunitybox.sh 
```


#FAQ
##uwsgi无法启动，报错mysqld.sock
###可能是mysql服务down了
```bash
sudo /etc/init.d/mysql restart 
#如果启动不要，请先kill wsgi相关的pid，否则被占用服务重启
```

###可能是服务器内存不足造成
该情况可以在wsgi运行的内存中发现提示

>spawned uWSGI master process (pid: 12149) fork(): Cannot allocate memory [core/master_utils.c line 726]

这情况就只能将云服务resize了。

###部署后二维码打印图片无法显示问题
问题可能为pillow的依赖lib不存在，具体可以看日志，如果为该情况，可以执行以下脚本去安装依赖，安装后记得重装pillow
```bash
sudo aptitude install libfreetype6-devpip 
uninstall Pillowpip 
install Pillow==2.5.1
```
###部署后，登录应用无法登录，后台报各种表找不到
按该章节操作[Django的setting配置](#django_setting)
