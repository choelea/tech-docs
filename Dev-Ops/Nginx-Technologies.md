---
Title: Nginx 问题收集
Description: Nginx 使用问题收集
---

总计收集在使用Nginx过程中遇见的问题。

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