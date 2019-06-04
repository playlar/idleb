# nginx配置
> 因为Blog这种低频，比较固定的，考虑直接生成静态页

## nginx.conf
> 本身nginx配置怎么放就不说了，你想独立放vhost去一个一个也可以，没差。

```
location ~ \.html$ {
    root html;
    index index.html;
    try_files $request_uri @idleb;
}

location @idleb {
    proxy_pass http://localhost:8080;
    proxy_connect_timeout 10;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header REMOTE-HOST $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}

location / {
    proxy_pass http://localhost:8080;
    proxy_connect_timeout 10;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header REMOTE-HOST $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```
## 配置文件解释
nginx的location从上向下依次匹配，基中@开头的location名称是nginx内容请求的，外部不可访问。

首先，以html接尾的请求，nginx首先在html目录下查找，如果找到就返回，如果找不到，就尝试请求@idleb的location路由，请求地址为$request_uri，这个参数也是客户端请求nginx的访问地址，@idleb就是通过反向代理访问后端。如果对于那些没有生成html页面放在html根目录下的资源来说，如js、css、图片等都会匹配到location /这个通配路由，这样就不致于某些资源无法访问。总是添加这个通用路由是个好习惯，当然，上面提到的js、css、图片等静态资源也可以放在html根目录下，修改下location ~ \.html$的规则就行，如location ~ \.(html|js|css|png)$，当然，你要将你后端tomcat下的静态资源存一份到nginx的html中就可以了。
