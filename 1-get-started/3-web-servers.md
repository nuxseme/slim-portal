# Web Servers

> It is typical to use the front-controller pattern to funnel appropriate HTTP requests received by your web server to a single PHP file. The instructions below explain how to tell your web server to send HTTP requests to your PHP front-controller file.

## PHP built-in server

```bash
cd public/
php -S localhost:8888
```

## Nginx configuration



```nginx
server {
    charset utf-8;
    client_max_body_size 128M;

  listen 80; ## listen for ipv4

  server_name server_name.com;
  root        /your/path;
  #index       web/index.html;

  access_log  /your/path/access.log;
  error_log  /your/path/error.log;

  location / {
      try_files /web/$uri /web/index.html 404;
  }

  location ~ \.(txt|xml|js|vue|css|ttf)$ {
      try_files /web/$uri /web/index.html 404;
      expires 7d;
  }

  location ~ \.(jpg|jpeg|gif|png|ico)$ {
      try_files /web/$uri /web/index.html 404;
      expires 30m;
  }

  location /protected {
      deny  all;
  }

  location ~ \.php$ {
     add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';

    add_header 'Access-Control-Allow-Origin' "$http_origin";
    add_header 'Access-Control-Allow-Credentials' "true";
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';

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
    	# 根据环境路由到不同的入口文件
  }

}
```

