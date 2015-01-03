#### 安装并配置supervisor

##### 安装

Ubuntu下

	apt-get install python-pip python-m2crypto supervisor

Centos下

	yum install python-pip python-m2crypto supervisor

supervisor安装后会有三个命令`supervisorctl supervisord echo_supervisord_conf`。其中supervisord是驻留服务Daemon，supervisorctl是控制台的程序，`echo_supervisord_conf`用于生成基本的配置文件。

##### 配置supervisor

首先生成基本配置文件

	echo_supervisord_conf > /etc/supervisord/supervisord.conf

######supervisord.conf文件说明

supervisord.conf文件最后有

	;[include]
	;files = relative/directory/*.ini

可以自定义添加的配置文件，去掉前面的`;`就可以起效，修改files后面的地址为你的相应配置文件地址。

	[include]
	files = /etc/supervisord.d/conf.d/*.ini

以后要添加由supervisor监管的程序，只要在以上目录下建立.ini文件就可以了，文件配置的格式如下：

	[program:shadowsocks]
	command=ssserver -c /etc/shadowsocks.json
	autorestart=true
	user=nobody

如果端口 < 1024，把上面的 user=nobody 改成 user=root。

##### **将supervisor加入自启动服务**

####Centos下自启动

在`/etc/init.d`下添加`supervisord`文件，如下：

		#!/bin/sh  
		#  
		# /etc/rc.d/init.d/supervisord  
		#  
		# Supervisor is a client/server system that  
		# allows its users to monitor and control a  
		# number of processes on UNIX-like operating  
		# systems.  
		#  
		# chkconfig: – 64 36  
		# description: Supervisor Server 
		# processname: supervisord
		# Source init functions  
		. /etc/init.d/functions  
		 	 
		RETVAL=0  
		prog=“supervisord”  
		pidfile=“/tmp/supervisord.pid”  
		lockfile=“/var/lock/subsys/supervisord”  
		  
		start()  
		{  
        echo -n $“Starting $prog: “  
        daemon –pidfile $pidfile supervisord -c /etc/supervisord.conf  
        RETVAL=$?  
        echo  
        [ $RETVAL -eq 0 ] && touch ${lockfile}  
		}  
		 	 
		stop()  
		{  
        echo -n $“Shutting down $prog: “  
        killproc -p ${pidfile} /usr/bin/supervisord  
        RETVAL=$?  
        echo  
        if [ $RETVAL -eq 0 ] ; then  
                rm -f ${lockfile} ${pidfile}  
        fi  
		}  
		  
		case “$1″ in  
		  
		start)  
		    start  
		  ;;  
		  
		  stop)  
		    stop  
		  ;;  
		  
		  status)  
	        status $prog  
		  ;;  
		  
		  restart)  
		    stop  
		    start  
		  ;;  
		  
		  *)  
		    echo “Usage: $0 {start|stop|restart|status}”  
		  ;;  
		  
		esac  

增加文件可执行属性：
		
		chmod 755 supervisord

添加到自启动服务：

		chkconfig supervisord on

#####ubuntu自启动服务

可从`https://github.com/Supervisor/initscripts`下载对应的脚本放到`/etc/init.d/`目录下

增加可运行属性：

		chmod 755 supervisord

添加到自启动服务：

		update-rc.d supervisord defaults

**注意：**这时要修改supervisor的配置文件`/etc/supervisord/supervisord.conf`，找到`pidfile=/tmp/supervisord.pid`，更改为`pidfile=/var/run/supervisord.pid`，不然supervisor总是启动不了。

#####运行supervisor
执行

	service supervisord start
	supervisorctl reload

可以用supervisorctl来显示目前supervisor监管的程序。比如：

	[root@bangwagon ~]#supervisorctl
	shadowsocks                      RUNNING   pid 10770, uptime 2 days, 18:35:56
	supernode                        RUNNING   pid 10769, uptime 2 days, 18:35:56
	supervisor> 

键入exit可退出。

此外，supervisor也可以通过web端来管理添加的监管程序。对supervisor.conf做一下修改，找到下面内容并修改为如下：

	[inet_http_server]         ; inet (TCP) server disabled by default
	port=0.0.0.0:9001        ; (ip_address:port specifier, *:port for all iface)
	username=user              ; (default is no username (open server))
	password=123               ; (default is no password (open server))

就可以用http://yourwebsite:9001打开，显示你的在supervisor下监管的程序。