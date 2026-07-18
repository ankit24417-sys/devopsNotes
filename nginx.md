               ## Nginx Webserver

> Nginx is a high performance web server that delivers websites,handle requests and can also as a reverse proxy,load balancer and caching tool to manage and scale web traffic efficiently

# Installation of nginx

> to install nginx use command => sudo apt install nginx
> some more command for nginx
> i) sudo systemctl start nginx
> ii) sudo systemctl stop nginx
> iii) sudo systemctl status nginx
> iv) sudo systemctl reload nginx
> v) sudo systemctl enable nginx
> vi) sudo systemctl disable nginx
> vii) sudo systemctl restart nginx

## Nginx file structure

[IMPORTANT]

# Before editing any server file always take backup of that file

````/
├── etc
│   └── nginx
│       ├── nginx.conf  => important one
│       ├── mime.types
│       ├── conf.d/
│       ├── snippets/
│       ├── sites-available/
│       ├── sites-enabled/
│       ├── modules-available/
│       └── modules-enabled/
├── var
│   ├── log
│   │   └── nginx
│   │       ├── access.log
│   │       └── error.log
│   ├── www
│   │   └── html/
│   │
│   ├── cache
│   │   └── nginx/
│   │
│   └── lib
│       └── nginx/
├── run
│   └── nginx.pid
├── usr
│   └── sbin
│       └── nginx
└── lib
    └── systemd
        └── system
            └── nginx.service
            ```
````

# Dealing with /etc/nginx/nginx.conf file

> basic structure of this file

```
  user;
   events{

   }
   http{
    server{
     location {

     }
    }
   }
```

#
