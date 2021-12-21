---
title: Nginx 知识点及问题收集
description: Nginx 使用问题收集
...

收集在使用Nginx过程中遇见的问题。

# 知识积累
## nginx安装
http://nginx.org/en/linux_packages.html#RHEL-CentOS
## nginx 查看信息
```
nginx -V
nginx -V 2>&1 | grep -o with-http_stub_status_module
```
## 负载均衡
平均负载示例如下; 以下配置必须保证两个实例都正常运行在，因为这个配置并不会failover。
```
upstream backend {
        server 127.0.0.1:8080;
        server 127.0.0.1:8081;
}
server {
        listen 80;
        server_name auth.jiu-shu.com;

        location / {
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://backend;
            client_max_body_size    10m;
        }
    }
```
> 上例中的8080 和 8081 端口都是Spring Boot的app。由于Java 是多线程的程序，在同一个虚拟机上运行多个实例并非最佳实践；这里只是方便测试。

## 反向代理的时候去掉前缀
 如下配置,访问`http://www.example.com/api/users/0` 和 `http://www.example.com/users/0` 将被代理至`http://localhost:3000/users/0`; 
```
server {
        listen 80;
        server_name www.example.com;

			location /api/ {
				proxy_set_header   Host             $host;
				proxy_set_header   x-forwarded-for  $remote_addr;
				proxy_set_header   X-Real-IP        $remote_addr;
				proxy_pass http://localhost:3000/;
			}
				location /user {
				proxy_set_header   Host             $host;
				proxy_set_header   x-forwarded-for  $remote_addr;
				proxy_set_header   X-Real-IP        $remote_addr;
				proxy_pass http://localhost:3000;
			}
}
```
## 使用GoAccess来实时监控
参考： https://goaccess.io/get-started
```
yum install goaccess // centos 安装方式
goaccess  /var/log/nginx/access.log -o /usr/share/nginx/html/report.html --log-format=COMBINED --real-time-html --daemonize // COMBINED适用于没有更改nginx日志格式；开启端口7890的websocket服务
```
>  注意开通对应的网络端口; daemonize 守护进程模式在目录`/var/log/nginx`下面执行
`goaccess access.log -o /usr/share/nginx/html/report.html --log-format=COMBINED --real-time-html --daemonize` 是不会成功的；不报错，但是进程没起来，原因不懂。 需要指定全路径。

配置nginx来访问http://*****/report.html 实时监控
## Nginx Steam 四层TCP 负载均衡
https://blog.csdn.net/michaelwoshi/article/details/97163777

日志没有测试成功
```
stream {
    upstream wcsbackend {
      hash $remote_addr consistent;
      server 36.110.117.58:8001;
    }
    proxy_timeout 30m;
		log_format basic '$remote_addr [$time_local] '    '$protocol $status $bytes_sent bytes_received '     '$session_time';
access_log /var/log/nginx/access-stream.log basic buffer=32k;    
server {
        listen 8001;
        proxy_pass wcsbackend;
    }
}

```

## Https 配置

下面网站可以申请打个域名的免费ssl证书试用及Nginx的配置
https://help.zerossl.com/hc/en-us/articles/360058295894-Installing-SSL-Certificate-on-NGINX


# 日志格式
http://nginx.org/en/docs/http/ngx_http_log_module.html
1.11.8及以上可以参考如下格式：
```
log_format logger-json-log escape=json '{' 
	'"@timestamp":"$time_iso8601",' 
	'"http_host":"$http_host",' 
	'"remote_addr":"$remote_addr",' 
	'"request_length":$request_length,' 
	'"request_method":"$request_method",' 
	'"request_uri":"$request_uri",' 
	'"request_time":$request_time,'
	'"server_name":"$server_name",' 
	'"status":$status,' 
	'"user_agent":"$http_user_agent",' 
	'"http_referer":"$http_referer",' 
	'"upstream_response_time":"$upstream_response_time",' 
	'"upstream_addr":"$upstream_addr",' 
	'"upstream_connect_time":"$upstream_connect_time"' 
'}'; 
```
# 日志分割 - logrotate
使用linux自带的logrotate来实现nginx 日志管理切割和压缩; 在创建文件/etc/logrotate.d/nginx，内容如下：
 
 ```
 /var/log/nginx/*log {
    create 0664 nginx root
    daily
    rotate 10
    missingok
    notifempty
    compress
    sharedscripts
    postrotate
        /bin/kill -USR1 `cat /run/nginx.pid 2>/dev/null` 2>/dev/null || true
    endscript
}
```
# 日志分割
nginx  本身并不支持日志rotating，需要自己准备类似如下的脚本，在凌晨设置对应的cron job来跑。
```
#!/bin/sh
#Rotate the nginx logs; and remove the logs of the month before last month
HIST_LOG_PATH=/var/log/nginx/history
CUR_LOG_PATH=/var/log/nginx
YESTERDAY=$(date -d "yesterday" +%y-%m-%d)
MONTH_TO_DELETE=$(date --date='-2 month' +'%y-%m')
mv ${CUR_LOG_PATH}/access.log ${HIST_LOG_PATH}/access_${YESTERDAY}.log
mv ${CUR_LOG_PATH}/error.log ${HIST_LOG_PATH}/error_${YESTERDAY}.log
## 向nginx 主进程发送USR1信号。 USR1信号是重新打开新日志文件
kill -USR1 $(cat /var/run/nginx.pid)
rm ${HIST_LOG_PATH}/access_${MONTH_TO_DELETE}*.log
rm ${HIST_LOG_PATH}/error_${MONTH_TO_DELETE}*.log
```

# 问题收集

### 反向代理后request的host和schema和浏览器请求不一致
反向代理后
下面如果不加proxy_set_header的两行，那么在microservice这个服务中，`request.getScheme() + "://" + request.getServerName()` 就会变成http://microservice.dev.com, nginx rewrite 之后，就可以获取到：http://www.dev.com
```
server_name  www.dev.com;
location / {
		 proxy_set_header Host $host;
  		 proxy_set_header X-Scheme $scheme;	
		 proxy_pass   http://microservice.dev.com:8091;
  	}
```

### bind() to 0.0.0.0:80 failed (98: Address already in use)
启动碰见以上问题，有两种可能

 1. 先检查80端口是否已经被其他http server占用 `sudo netstat -nlpt`
 2. remove the IPv6 bind block (something along the lines of ::1:80。 参考：http://serverfault.com/questions/520535/nginx-is-still-on-port-80-bind-to-0-0-0-080-failed-98-address-already-in
 
### 403 forbidden (13: Permission denied)
参考：[Nginx报错403 forbidden (13: Permission denied)的解决办法](https://www.hi-docs.com/article/detail-MTE1.html)
解决办法一： 关闭 SELinux  （在了解了SELinux的重要性后，决定继续寻找更好的解决办法）

需要进一步了解SELinux相关，需要解决办法二：（感谢Zeal老师给出的解决方案）
> Every directory has a SeLinux context and the default 'Document Root' ( /var/www/html ) has an context which allows the nginx / apache user to access the directory.
The new ROOT ( /data/images ) will not have the same context and thus SeLinux  is blocking the access.
You can verify with ls -lZ /Default-Document-Root and verify the context and associate the same context to /data/images.
This should ideally solve the issue, can you try and verify once  :- 
`chcon -R -u system_u -t httpd_sys_content_t /data/`

相信ftp等服务，如果更改了根目录，也会有同样的问题。需要更深入的对SELinux学习。

#### (13: Permission denied) while connecting to upstream:[nginx]

https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx

####  *48 stat() "/home/nantong/www/index.html" failed (13: Permission denied),

Nginx在目录中运行，因此，如果您无法从nginx用户cd到该目录，（日志中的stat命令也是如此）。 确保nginx的用户可以一直CD到`/home/nantong/www`。 您可以通过运行`sudo -u nginx stat /home/nantong/html`来确认该统计信息将失败还是成功
SELinux已经禁用了；查看目录权限如下：
```
[nantong@localhost ~]$ ls -l /home/nantong
total 4
drwxrwxr-x 4 nantong nantong 4096 Sep 28 15:37 www
```
```
gpasswd -a nginx nantong  //nginx运行用户可以从nginx.conf 查看
```
如果依然报错， 可能是无法进入/home/nantong; 通过运行`sudo chmod g+x /home/nantong` 来解决此问题。