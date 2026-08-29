# 🌐 Complete Nginx Web Server Guide (Beginner to Advanced)

> **Nginx** (pronounced *"Engine-X"*) is an open-source, high-performance HTTP web server, reverse proxy, load balancer, HTTP cache, and API gateway. It is designed to handle thousands of concurrent connections with low memory usage and high efficiency.

---

## 📌 Table of Contents
1. [Overview & Architecture](#-overview--architecture)
2. [Installation & Service Management](#-installation--service-management)
3. [Nginx File & Directory Structure](#-nginx-file--directory-structure)
4. [Master Configuration (`nginx.conf`) Anatomy](#-master-configuration-nginxconf-anatomy)
5. [Understanding Nginx Contexts & Directives](#-understanding-nginx-contexts--directives)
6. [Location Block Matching Rules](#-location-block-matching-rules)
7. [Practical Use-Cases & Configurations](#-practical-use-cases--configurations)
   - [Hosting a Static Website](#1-hosting-a-static-website)
   - [Setting Up a Reverse Proxy](#2-setting-up-a-reverse-proxy)
   - [Configuring a Load Balancer](#3-configuring-a-load-balancer)
   - [Enabling HTTPS / SSL (Certbot / Let's Encrypt)](#4-enabling-https--ssl-lets-encrypt)
8. [Security & Performance Optimization](#-security--performance-optimization)
9. [Troubleshooting & Log Analysis](#-troubleshooting--log-analysis)
10. [Nginx Quick Reference Cheat Sheet](#-nginx-quick-reference-cheat-sheet)

---

## 🚀 Overview & Architecture

### What makes Nginx different?
Unlike traditional web servers (like Apache) that create a new thread or process for every incoming request, Nginx uses an **event-driven, asynchronous, non-blocking architecture**.

```
Client Requests ---> [ Master Process ]
                         |
           +-------------+-------------+
           |                           |
   [ Worker Process 1 ]        [ Worker Process 2 ]
   (Handles 1000s of           (Handles 1000s of
   connections via Event Loop)  connections via Event Loop)
```

- **Master Process**: Reads configuration, validates syntax, binds to ports, and manages worker processes.
- **Worker Processes**: Handle actual network connections, read requests, write responses, and interact with upstream servers.

---

## 📥 Installation & Service Management

### 1. Installing Nginx (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install -y nginx
```

### 2. Service Commands (`systemctl`)
```bash
# Check status of Nginx service
sudo systemctl status nginx

# Start Nginx
sudo systemctl start nginx

# Stop Nginx
sudo systemctl stop nginx

# Restart Nginx (completely stops and starts server)
sudo systemctl restart nginx

# Reload Nginx (reloads configuration gracefully WITHOUT dropping live connections)
sudo systemctl reload nginx

# Enable Nginx to automatically start on boot
sudo systemctl enable nginx

# Disable Nginx from auto-starting on boot
sudo systemctl disable nginx
```

> [!IMPORTANT]
> **ALWAYS test configuration syntax before reloading or restarting Nginx!**
> ```bash
> sudo nginx -t
> ```
> If syntax is ok, you will see `syntax is ok` / `test is successful`. Only then proceed to `sudo systemctl reload nginx`.

### 3. Firewall Setup (UFW)
```bash
# Allow Nginx traffic through firewall
sudo ufw allow 'Nginx Full'

# Verify UFW status
sudo ufw status
```

---

## 📁 Nginx File & Directory Structure

Here is a breakdown of the standard Nginx directory layout on Linux systems:

```
/etc/nginx/                     # Main configuration directory
├── nginx.conf                  # Core configuration file (Global settings)
├── mime.types                  # Map of file extensions to MIME types (e.g. .html -> text/html)
├── conf.d/                     # Additional config drop-ins (*.conf)
├── snippets/                   # Reusable configuration snippets (SSL settings, security headers)
├── sites-available/            # All site configuration files (active & inactive)
├── sites-enabled/              # Symbolic links to active sites in sites-available/
├── modules-available/          # Available dynamic modules
└── modules-enabled/            # Enabled dynamic modules

/var/www/                       # Web root directory
└── html/                       # Default web page directory (index.nginx-debian.html)

/var/log/nginx/                 # Log directory
├── access.log                  # Records every incoming request
└── error.log                   # Records errors, warnings, and diagnostic information

/run/nginx.pid                  # File storing Process ID (PID) of running Nginx master process
/usr/sbin/nginx                 # Main Nginx executable binary
```

> [!TIP]
> **Why `sites-available` vs `sites-enabled`?**
> - **`sites-available/`**: Contains actual configuration files for all your websites.
> - **`sites-enabled/`**: Contains symbolic links (shortcuts) pointing to files in `sites-available/`.
> 
> To enable a site:
> ```bash
> sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/
> ```
> To disable a site:
> ```bash
> sudo rm /etc/nginx/sites-enabled/mywebsite
> ```
> *(Removing the symlink disables the website without deleting your original config file!)*

---

## ⚙️ Master Configuration (`nginx.conf`) Anatomy

The `nginx.conf` file uses a tree-like block structure called **Contexts**. Each block contains **Directives** (key-value instructions ending with `;`).

```nginx
# 1. GLOBAL / MAIN CONTEXT
user www-data;                  # User account under which worker processes run
worker_processes auto;          # Number of worker processes (auto = CPU cores count)
pid /run/nginx.pid;             # Location of PID file
include /etc/nginx/modules-enabled/*.conf;

# 2. EVENTS CONTEXT (Network connection settings)
events {
    worker_connections 1024;    # Max simultaneous connections per worker process
    # total connections = worker_processes * worker_connections
}

# 3. HTTP CONTEXT (All HTTP/HTTPS related configuration)
http {
    # Basic Settings
    sendfile on;                # Enables efficient file transfer directly from disk to socket
    tcp_nopush on;              # Optimizes payload sending
    tcp_nodelay on;
    keepalive_timeout 65;       # Timeout for client keep-alive connection (seconds)
    types_hash_max_size 2048;
    server_tokens off;          # Hides Nginx version for security

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging Settings
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Compression Settings
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml;

    # Virtual Host Configs (Include all configs from conf.d and sites-enabled)
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## 🔍 Understanding Nginx Contexts & Directives

Nginx configuration is organized into nested blocks called **Contexts**:

| Context | Purpose | Scope |
| :--- | :--- | :--- |
| **Main (Global)** | Configures global Nginx process settings (user, worker count, PID) | Top-level |
| **`events`** | Configures connection handling and networking loop parameters | Inside Main |
| **`http`** | Configures global web server parameters (MIME, logging, gzip, timeouts) | Inside Main |
| **`server`** | Defines a virtual host / domain (port, domain name, SSL) | Inside `http` |
| **`location`** | Defines how to process requests matching specific URI paths | Inside `server` |
| **`upstream`** | Defines a group of backend servers for proxying & load balancing | Inside `http` |

---

## 🎯 Location Block Matching Rules

A `location` block matches the URI (path) of an incoming HTTP request. Nginx evaluates location blocks using specific modifier symbols with strict priorities:

```nginx
location [modifier] [URI pattern] {
    # Directives for matching URIs
}
```

### Modifier Types & Matching Hierarchy

| Modifier | Match Type | Priority | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `=` | Exact Match | **1 (Highest)** | Matches the exact URI path strictly. | `location = /login { ... }` |
| `^~` | Preferential Prefix | **2** | If longest matching prefix has `^~`, stop regex checking. | `location ^~ /images/ { ... }` |
| `~` | Case-Sensitive Regex | **3** | Matches URI using regular expression (case-sensitive). | `location ~ \.php$ { ... }` |
| `~*` | Case-Insensitive Regex| **3** | Matches URI using regex (case-insensitive). | `location ~* \.(png|jpg|css)$` |
| *(none)* | Standard Prefix | **4 (Lowest)** | Matches any URI starting with this prefix. | `location /blog { ... }` |

### Example matching rule in practice:
```nginx
server {
    listen 80;
    server_name example.com;

    # Matches ONLY http://example.com/about exactly
    location = /about {
        return 200 "Exact match for /about";
    }

    # Matches any request starting with /images/ and skips regex check
    location ^~ /images/ {
        root /var/www/media;
    }

    # Case-insensitive match for static asset extensions
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public";
    }

    # Default fallback match for any other URI
    location / {
        root /var/www/html;
        index index.html index.htm;
        try_files $uri $uri/ =404;
    }
}
```

---

## 🚀 Practical Use-Cases & Configurations

### 1. Hosting a Static Website

Create file `/etc/nginx/sites-available/mywebsite`:

```nginx
server {
    listen 80;                          # Port to listen on (HTTP)
    server_name mywebsite.com www.mywebsite.com; # Domain names

    root /var/www/mywebsite;            # Root folder containing index.html
    index index.html index.htm;         # Default files to serve

    # Handle static requests
    location / {
        # try_files checks if file exists ($uri), then folder ($uri/), else returns 404
        try_files $uri $uri/ =404;
    }

    # Custom 404 error page
    error_page 404 /custom_404.html;
    location = /custom_404.html {
        root /var/www/mywebsite;
        internal;                        # Can only be called internally by Nginx
    }
}
```

**Steps to enable:**
```bash
sudo mkdir -p /var/www/mywebsite
sudo chown -R $USER:$USER /var/www/mywebsite
echo "<h1>Hello from My Nginx Website!</h1>" > /var/www/mywebsite/index.html

sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### 2. Setting Up a Reverse Proxy

A **Reverse Proxy** acts as an intermediary, receiving requests from clients and forwarding them to backend applications (e.g. Node.js, Express, React, Python/Django, Go, Docker containers).

```
Client  --->  [ Nginx Server (Port 80/443) ]  --->  [ Backend App (127.0.0.1:3000) ]
```

Create file `/etc/nginx/sites-available/app-proxy`:

```nginx
server {
    listen 80;
    server_name app.example.com;

    location / {
        # Forward requests to internal app running on port 3000
        proxy_pass http://127.0.0.1:3000;

        # Standard Proxy Headers (preserve client request details)
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support (if your application uses WebSockets/Socket.io)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Timeout settings
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

> [!NOTE]
> **Why do we need Proxy Headers?**
> Without `X-Real-IP` and `X-Forwarded-For`, your backend application would see every incoming user's IP address as `127.0.0.1` (the IP of Nginx), losing the real client IP info!

---

### 3. Configuring a Load Balancer

Nginx can distribute client requests across multiple backend servers to ensure high availability and reliability.

```
                         +---> [ Backend Server 1: 10.0.0.1:8080 ]
                         |
Client ---> [ Nginx ] ---+---> [ Backend Server 2: 10.0.0.2:8080 ]
                         |
                         +---> [ Backend Server 3: 10.0.0.3:8080 ]
```

Create file `/etc/nginx/sites-available/load-balancer`:

```nginx
# Define the pool of backend servers in an upstream block
upstream backend_servers {
    # Load balancing algorithms (Uncomment one if needed):
    # least_conn; # Send request to server with fewest active connections
    # ip_hash;    # Send request based on client IP (session persistence)

    server 10.0.0.1:8080 weight=3; # Receives 3x more traffic (Weighted Round Robin)
    server 10.0.0.2:8080;
    server 10.0.0.3:8080 backup;   # Only used when primary servers are down
}

server {
    listen 80;
    server_name service.example.com;

    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

### 4. Enabling HTTPS / SSL (Let's Encrypt & Certbot)

Encrypting website traffic with SSL/TLS certificates is essential. Certbot automates SSL certificate acquisition and configuration for Nginx.

```bash
# 1. Install Certbot and the Nginx plugin
sudo apt update
sudo apt install -y certbot python3-certbot-nginx

# 2. Automatically request certificate and configure Nginx
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot automatically modifies your Nginx configuration to add SSL directives:

```nginx
server {
    listen 443 ssl http2; # SSL enabled with HTTP/2 support
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    root /var/www/html;
    index index.html;
}

# HTTP to HTTPS automatic redirection server block
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}
```

---

## 🛡️ Security & Performance Optimization

### 1. Security Enhancements
```nginx
# 1. Hide Nginx Version (Add in http block of nginx.conf)
server_tokens off;

# 2. Block access to hidden files (e.g. .git, .env, .htaccess)
location ~ /\. {
    deny all;
    access_log off;
    log_not_found off;
}

# 3. Add Security Headers (Add inside server block)
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "no-referrer-when-downgrade" always;

# 4. Limit Request Rate (DDoS Protection)
# In http block:
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

# In location block:
limit_req zone=one burst=20 nodelay;
```

### 2. Performance Tuning
```nginx
# 1. Static Asset Caching
location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|woff|woff2|ttf)$ {
    expires 30d;
    add_header Cache-Control "public, no-transform";
    access_log off;
}

# 2. Enable Gzip Compression (In http block)
gzip on;
gzip_comp_level 5;
gzip_min_length 256;
gzip_proxied any;
gzip_types
    text/plain
    text/css
    application/json
    application/javascript
    text/xml
    application/xml
    image/svg+xml;
```

---

## 🛠️ Troubleshooting & Log Analysis

When Nginx exhibits issues or returns HTTP errors, inspect the logs immediately!

### Reading Log Files
```bash
# View real-time error log entries
sudo tail -f /var/log/nginx/error.log

# View real-time access log entries
sudo tail -f /var/log/nginx/access.log

# Search for 50x errors in access log
grep "HTTP/1.1\" 50" /var/log/nginx/access.log
```

### Common Nginx HTTP Status Codes & Fixes

| Status Code | Meaning | Common Cause & Resolution |
| :--- | :--- | :--- |
| **403 Forbidden** | Client forbidden from accessing path | Incorrect file permissions on `root` directory (`chmod`/`chown`), or directory listing disabled (`autoindex off`). |
| **404 Not Found** | Resource missing | File path does not exist under `root`, or `try_files` path mismatch. |
| **502 Bad Gateway** | Nginx failed to reach backend app | Upstream server (Node.js/Python/PHP-FPM) is offline or not listening on specified port/socket. |
| **503 Service Unavailable** | Server temporary overload | Backends overloaded or rate-limiting (`limit_req`) triggered. |
| **504 Gateway Timeout** | Backend took too long to answer | Upstream application hung or slow query execution. Increase `proxy_read_timeout`. |

---

## ⚡ Nginx Quick Reference Cheat Sheet

| Command / Action | Syntax / Location |
| :--- | :--- |
| **Test Configuration** | `sudo nginx -t` |
| **Graceful Reload** | `sudo systemctl reload nginx` |
| **Check Active Config** | `nginx -T` |
| **Main Config File** | `/etc/nginx/nginx.conf` |
| **Sites Storage** | `/etc/nginx/sites-available/` |
| **Active Sites Symlinks** | `/etc/nginx/sites-enabled/` |
| **Default Web Root** | `/var/www/html/` |
| **Access Log** | `/var/log/nginx/access.log` |
| **Error Log** | `/var/log/nginx/error.log` |
| **Create Symlink** | `sudo ln -s /etc/nginx/sites-available/<site> /etc/nginx/sites-enabled/` |
| **Remove Symlink** | `sudo rm /etc/nginx/sites-enabled/<site>` |
