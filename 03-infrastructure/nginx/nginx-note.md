# Nginx
```java
// nginx.conf
server {
        listen       7777;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   "C:\myNginx";
            index  index.html index.htm;
        }
```
<empty-block/>
<empty-block/>
<empty-block/>
<empty-block/>
# 색인과 출처
- nginx 설치 및 구동 : [https://www.youtube.com/watch?v=Sc0nJtVtWSI](https://www.youtube.com/watch?v=Sc0nJtVtWSI)
- 설정 : [https://grandj.tistory.com/246](https://grandj.tistory.com/246)
- [https://mrgalbi.blogspot.com/2016/08/nginx.html](https://mrgalbi.blogspot.com/2016/08/nginx.html)
- [https://kimjongmo.github.io/install/nginx](https://kimjongmo.github.io/install/nginx)
- web server와 WAS : [https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html](https://gmlwjd9405.github.io/2018/10/27/webserver-vs-was.html)
