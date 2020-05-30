# Example



## 环境配置

1  php > 7 

2  nginx

3  composer 

4  phpunit

## 初始化项目

```bash
composer create-project slim/slim-skeleton:dev-master [my-app-name]
```

## Web Server

```nginx
log_format access_log_format '|$remote_addr|$remote_user|$time_local|$http_host|$request|$status|$body_bytes_sent|$http_referer|$http_user_agent|$request_time|$proxy_add_x_forwarded_for|$upstream_addr|$cookie_clientId|$cookie_userId|$cookie_pskey|$cookie_fingerprint|$http_x_forwarded_for|';
server {
    charset utf-8;
    client_max_body_size 128M;

    listen       80;

    server_name 127.0.0.1;
    root        path;
#    index       /web/index.html;

    #log_format test_k_access_log_443 '|$remote_addr|$remote_user|$time_local|$http_host|$request|$status|$body_bytes_sent|$http_referer|$http_user_agent|$request_time|$proxy_add_x_forwarded_for|$upstream_addr|$cookie_clientId|$cookie_userId|$cookie_pskey|$cookie_fingerprint|$http_x_forwarded_for|';
    access_log  path/runtime/access.log access_log_format;
    error_log  path/runtime/error.log;


    location / {
         try_files /web/$uri /web/index.html 404;
    }
    location ~ \.(jpg|jpeg|gif|png|ico)$ {
        try_files /web/$uri  404;
        expires 30m;
    }

    location ~ \.(txt|xml|js|vue|css|ttf|json)$ {
        try_files /web/$uri  404;
        expires 7d;
    }


    location /protected {
        deny  all;
    }

    location ~ \.php$ {
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type' always;
        add_header 'Access-Control-Allow-Origin' "$http_origin" always;
        add_header 'Access-Control-Allow-Credentials' "true" always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;


        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/$fastcgi_script_name;
        fastcgi_param REQUEST_URI $php_url?$args;
        fastcgi_param REAL_REQUEST_URI $request_uri;

    fastcgi_pass   127.0.0.1:9000;
        try_files $uri =404;
    }

    location ~ /\.(ht|svn|git) {
        deny all;
    }

    location ~* ^/api(/.*)$ {
        set $php_url $1;
        try_files $uri $uri/ /index-test.php?$args;
    }
}
```



## 构建项目 

```
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── app   //默认配置目录 
│   ├── dependencies.php  //设置依赖 比如日志  邮件  短信
│   ├── middleware.php  //中间件配置
│   ├── repositories.php //仓库依赖
│   ├── routes.php //路由设置
│   └── settings.php  //设置项
├── composer.json
├── composer.lock
├── docker-compose.yml
├── logs
│   └── README.md
├── phpstan.neon.dist
├── phpunit.xml
├── public  //入口文件 
│   ├── index-test.php  //测试环境配置
│   ├── index.php  
│   └── web
├── runtime
│   ├── access.log
│   └── error.log
├── src
│   ├── Application  //应用层
│   ├── Domain  //领域层
│   └── Infrastructure //基础设施层 
├── test.php
├── tests  //单元测试
│   ├── Application
│   ├── Domain
│   ├── Infrastructure
│   ├── TestCase.php
│   └── bootstrap.php
├── var
│   └── cache
└── vendor  //第三方库
```



Application的写法可以自己定义，Slim官方的例子是采用Action这种粒度的接口，个人还是喜欢Controller这种聚合粒度

Slim官方采用的DDD架构目录模式也可以变化下

Domain 存储模型和依赖的仓库接口

Application 放置Controller Action,Handler,Middleware ...

Infrastructure 放置持久化设施，邮件服务，pdf生成器



