# nginx django 配置

## 安装nginx
对于Ubuntu可以使用  sudo apt-get install nginx
开启服务:service nginx start
停止服务:service nginx stop
重启服务:service nginx restart

## 安装uwsgi
使用pip进行安装

## 配置nginx
1.进入 /etc/nginx/sites-available 创建 mysite.conf
内容如下
```python
server {
    listen 80;
    server_name mysite;
    charset utf-8;

    client_max_body_size 75M;

    location /static {
        alias /root/static;
    }
    #静态文件目录
    location /media {
        alias /root/media;
    }
    #上传文件目录
    location / {
        uwsgi_pass 127.0.0.1:8001;
        include /etc/nginx/uwsgi_params;
    }

}
```
2.进入 /etc/nginx/sites-enabled 删除default
3.创建软链接 `ln -s /etc/nginx/sites-available/mysite.conf /etc/nginx/sites-enabled/mysite.conf` 

## 配置uwsgi
1.找个地方创建一个用来存放uwsgi.ini的文件夹（我的django project是nginx_test，推荐在django project的同级目录）
2.创建mysite.ini,内容如下
```python
[uwsgi]
chdir=/root/myweb/nginx_test
module=nginx_test.wsgi:application
master=True
processes=4
harakiri=60
max_requests=5000
socket=127.0.0.1:8001

pidfile=/root/myweb/uwsgis/master.pid
vacuum=True
daemonize=/root/myweb/uwsgis/nginx_test.log
```
###我的root下的文件树
```
├── media                                                    #上传文件目录
├── myweb                                                    #存放django项目
│   ├── nginx_test                                           #测试用django项目
│   │   ├── manage.py
│   │   └── nginx_test
│   │       ├── __init__.py
│   │       ├── __pycache__
│   │       │   ├── __init__.cpython-35.pyc
│   │       │   ├── settings.cpython-35.pyc
│   │       │   ├── urls.cpython-35.pyc
│   │       │   └── wsgi.cpython-35.pyc
│   │       ├── settings.py
│   │       ├── urls.py
│   │       └── wsgi.py
│   └── uwsgis                                               #存放uwsgi配置文件
│       ├── master.pid
│       ├── mysite.ini
│       └── nginx_test.log
└── static                                                   #静态文件目录
    └── aaa.txt
```

## 开启服务
1.`uwsgi --ini /root/myweb/uwsgis/mysite.ini`
2.`service nginx start`
3.直接访问localhost或者ip地址就可以啦，因为nginx监听的80端口。

### ps.
1.在改变了django项目中的某些东西后必须重启nginx服务（`service nginx restart`）
2.如果访问不到静态文件，将 /etc/nginx/nginx.conf 中第一行改为`user root;`然后重启服务。
3.参考视频  <https://www.bilibili.com/video/av31456425/>
4.修改django内容后需要杀死uwsgi进程然后在重启uwsgi。