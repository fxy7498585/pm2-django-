使用pm2守护Django进程的想法由来

自己在本地使用django做了个后台管理系统，想部署到服务器，可是静态文件访问不到 ，便百度找到了些方法，使用nginx+uwsgi部署，
自己便试了下，出现些问题
在已经使用nginx代理的情况下：
1.使用uwsgi启动Django项目，静态文件还是访问不到
2.使用python manage.py runserver 启动项目是可以访问到静态文件
 
不知道为什么第一种情况不好使，后面深入学习下，看看能不能找到原因


---uwsgi配置

django_uwsgi.ini
[uwsgi]

chdir = /root/django/demo

module = demo.wsgi

socket = http://127.0.0.1:8000

master = True

processes = 4

threads = 1


vacuum = True


#backend run uwsgi

daemonize = %(chdir)/log/uwsgi-8000.log

log-maxsize = 1024*1024*1024

pidfile = %(chdir)/pid/uwsgi-8000.pid



----nginx配置   /www/server/panel/vhost/nginx
nginx_uwsgi.conf

server
    {
    	listen 80;
    	server_name www.wangjiapets.cn;
      	charset utf-8;
    
    	location / {
      		proxy_pass http://127.0.0.1:8000;
    	}
        location static {
        	alias /root/django/demo/static;
    	}
  	}

目前可以确认的是反向代理没有配置错误 （别打脸）



只能暂时放弃uwsgi 启动 使用了pm2启动进程


/root/django/demo 
在项目根目录配置
start.sh
    python manage.py runserver

执行 chmod +x start.sh  //添加执行权限

start.json
    {
  "apps":
    {
      "name": "djangoDemo", //应用程序的名称
      "cwd": "/root/django/demo/", //应用程序所在的目录
      "script": "./start.sh", //应用程序的脚本路径
      "exec_interpreter": "bash", //应用程序的脚本类型，这里使用的shell，默认是nodejs
      "min_uptime": "60s", //最小运行时间，这里设置的是60s即如果应用程序在60s内退出，pm2会认为程序异常退出，此时触发重启max_restarts设置数量
      "max_restarts": 30, //设置应用程序异常退出重启的次数，默认15次（从0开始计数）
      "exec_mode" : "fork", //应用程序启动模式，默认是fork
      "error_file" : "./test-err.log", //自定义应用程序的错误日志文件
      "out_file": "./test-out.log", //自定义应用程序日志文件
      "pid_file": "./test.pid", //自定义应用程序的pid文件
      "watch": true //是否启用监控模式，默认是false。如果设置成true，当应用程序变动时，pm2会自动重载。这里也可以设置你要监控的文件。
    }
}


执行pm2 start start.json

项目可以成功启动







