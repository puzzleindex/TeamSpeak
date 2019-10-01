# TeamSpeak
#安装说明  
我用的系统是CentOS7，其他系统的安装方式类似。需要root权限，以下均以root账户输入。  

#更新系统和依赖  
'yum -y update'  
'yum -y install nano wget perl tar net-tools bzip2'  
添加用户安装TS  
使用普通用户权限运行TeamSpeak，隔离ts服务端与其他系统服务  

adduser teamspeak  
passwd teamspeak  
会提示为新建的用户设置密码，这个密码之后不会用到。  
坑点：因为国内ts1汉化的客户端版本不是最新版，不能支持新版服务端，故选择老版本服务端安装。  

wget https://github.com/puzzleindex/TeamSpeak/raw/master/teamspeak3-server_linux_amd64-3.0.13.6.tar.bz2  
tar xvf teamspeak3-server_linux_amd64-3.0.13.6.tar.bz2  
cd teamspeak3-server_linux_amd64  
cp * -R /home/teamspeak  
cd ..
rm -rf teamspeak3-server_linux_amd64*  
chown -R teamspeak:teamspeak /home/teamspeak  
设置TS服务和防火墙  
nano /lib/systemd/system/teamspeak.service  
在打开的文件编辑器里输入  

[Unit]  
Description=Team Speak 3 Server  
After=network.target  
[Service]  
WorkingDirectory=/home/teamspeak/  
User=teamspeak  
Group=teamspeak  
Type=forking  
ExecStart=/home/teamspeak/ts3server_startscript.sh start inifile=ts3server.ini  
ExecStop=/home/teamspeak/ts3server_startscript.sh stop  
PIDFile=/home/teamspeak/ts3server.pid  
RestartSec=15  
Restart=always  
[Install]  
WantedBy=multi-user.target  
Ctrl+O回车保存，Ctrl+E退出  

如果你的服务器打开了防火墙，就需要添加如下规则，对CentOS7上的firewall使用如下命令  

firewall-cmd --zone=public --add-port=9987/udp --permanent  
firewall-cmd --zone=public --add-port=10011/tcp --permanent  
firewall-cmd --zone=public --add-port=30033/tcp --permanent  
firewall-cmd --reload  
启动TS服务  
systemctl start teamspeak  
systemctl enable teamspeak  
确保TeamSpeak服务正确运行  
  
systemctl status teamspeak  
管理员登陆TS服务器  
TS服务器第一次运行时，会生成一个一次性的权限密钥，用于给你本地端设置管理员权限。  

cat /home/teamspeak/logs/ts3server_*  
如果TS服务器正确运行，你应该能看到类似如下的输出  

2018-04-24 05:15:19.808720|INFO    |ServerLibPriv |   |TeamSpeak 3 Server 3.0.13.6 (2016-11-08 08:48:33)  
2018-04-24 05:15:19.808834|INFO    |ServerLibPriv |   |SystemInformation: Linux 2.6.32-042stab126.2 #1 SMP Wed Dec 6 18:08:29 MSK 2017   x86_64 Binary: 64bit  
2018-04-24 05:15:19.808873|WARNING |ServerLibPriv |   |The system locale is set to "C" this can cause unexpected behavior. We advice you  to repair your locale!  
2018-04-24 05:15:19.808898|INFO    |ServerLibPriv |   |Using hardware aes  
2018-04-24 05:15:19.809882|INFO    |DatabaseQuery |   |dbPlugin name:    SQLite3 plugin, Version 3, (c)TeamSpeak Systems GmbH  
2018-04-24 05:15:19.819093|INFO    |DatabaseQuery |   |dbPlugin version: 3.11.1  
2018-04-24 05:15:19.819377|INFO    |DatabaseQuery |   |checking database integrity (may take a while)  
2018-04-24 05:15:19.835790|INFO    |SQL           |   |db_CreateTables() tables created  
2018-04-24 05:15:19.880463|WARNING |Accounting    |   |Unable to find valid license key, falling back to limited functionality  
2018-04-24 05:15:22.175884|INFO    |              |   |Puzzle precompute time: 2264  
2018-04-24 05:15:22.177343|INFO    |FileManager   |   |listening on 0.0.0.0:30033, :::30033  
2018-04-24 05:15:22.178288|INFO    |VirtualSvrMgr |   |executing monthly interval  
2018-04-24 05:15:22.178463|INFO    |VirtualSvrMgr |   |reset virtualserver traffic statistics  
2018-04-24 05:15:22.200638|INFO    |CIDRManager   |   |updated query_ip_whitelist ips: 127.0.0.1/32, ::1/128,  
2018-04-24 05:15:22.200965|INFO    |Query         |   |listening on 0.0.0.0:10011, :::10011  
2018-04-24 05:25:30.593174|INFO    |ServerMain    |   |Received signal SIGTERM, shutting down.  
2018-04-24 05:15:22.198477|INFO    |VirtualServer |1  |listening on 0.0.0.0:9987, :::9987  
2018-04-24 05:15:22.199370|WARNING |VirtualServer |1  |--------------------------------------------------------  
2018-04-24 05:15:22.199410|WARNING |VirtualServer |1  |ServerAdmin privilege key created, please use the line below  
2018-04-24 05:15:22.199434|WARNING |VirtualServer |1  |token=SVA+zpAPeqAr5QtJ2pSY0pTf5TwSRuPv4vuoDUdf  
2018-04-24 05:15:22.199458|WARNING |VirtualServer |1  |--------------------------------------------------------  
你应该能在最后几行看到ServerAdmin privilege key，登陆ts服务器，客户端输入token=后面的privilege key，你就是服务器管理员了。  
