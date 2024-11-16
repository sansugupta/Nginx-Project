# Comprehensive Nginx Guide

# 📌 Project Highlights:

Core Infrastructure Setup


Implemented NGINX as a reverse proxy and web server
Configured static content serving with proper MIME types
Established robust server directives and location blocks
Set up systematic file structure for optimal content management


Advanced Implementation


Created sophisticated location blocks with:
• Custom routing rules
• Dynamic file serving
• Intelligent try_files directives
• Regular expression pattern matching
Implemented URL redirects (307) and rewrites for optimal routing.


Load Balancing Architecture


Designed and implemented a Round Robin load balancing system
Set up multiple backend servers using Docker containers
Created a Node.js test application to demonstrate load distribution
Implemented health checks and server monitoring
Developed a bash testing script for real-time load balancer verification.

## Table of Contents
- [Introduction](#introduction)
- [Installation and Setup](#installation-and-setup)
- [Core Concepts](#core-concepts)
- [Configuration Guide](#configuration-guide)
- [Location Blocks Deep Dive](#location-blocks-deep-dive)
- [Advanced Features](#advanced-features)
- [Load Balancing](#load-balancing)
- [Real-World Examples](#real-world-examples)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

## Introduction

### What is Nginx?
Nginx is a powerful, high-performance web server that serves multiple purposes:

* Web Server: Serves static and dynamic content
* Reverse Proxy: Acts as an intermediary for requests from clients
* Load Balancer: Distributes incoming traffic across multiple servers
* SSL/TLS Terminator: Handles encryption and security

### Key Features

* Event-driven architecture
* Asynchronous, non-blocking I/O operations
* Low memory footprint
* High concurrency capability
* Modular design

## Installation and Setup

### Ubuntu/Debian Installation
```bash
# Update system packages
sudo apt-get update

# Install Nginx
sudo apt-get install nginx

# Start Nginx service
sudo systemctl start nginx

# Enable auto-start on boot
sudo systemctl enable nginx

# Verify installation
nginx -v
```

### Basic System Commands
```bash
# Check Nginx status
sudo systemctl status nginx

# Stop Nginx
sudo systemctl stop nginx

# Restart Nginx
sudo systemctl restart nginx

# Reload configuration without downtime
sudo systemctl reload nginx

# Test configuration
sudo nginx -t
```

## Core Concepts

### Directory Structure
```plaintext
/etc/nginx/
├── nginx.conf     # Main configuration file
├── conf.d/        # Custom configuration files
├── sites-available/  # Available site configurations
├── sites-enabled/   # Enabled site configurations (symlinks)
├── mime.types     # MIME type mappings
└── modules-enabled/ # Enabled modules
```

### Basic Configuration Example
```nginx
# Main context
user www-data;
worker_processes auto;

events {
    worker_connections 768;
}

http {
    # Include MIME types
    include mime.types;
    
    # Basic settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    
    # Server block
    server {
        listen 8080;
        root /home/sanskar/mysite;
        
        # Basic location block
        location / {
            try_files $uri $uri/ =404;
        }
    }
}
```

## Configuration Guide

### Serving Static Content

#### Project Structure
```plaintext
/home/sanskar/mysite/
├── index.html
├── styles.css
└── images/
    └── logo.png
```

#### HTML Example
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8"/>
        <title>My Nginx Project</title>
        <link rel="stylesheet" href="./styles.css">
    </head>
    <body>
        <h1>Hello my friends. I am served from NGINX</h1>
        <img src="./images/logo.png" alt="Logo">
    </body>
</html>
```

#### CSS Example
```css
h1 {
    background-color: pink;
    color: aqua;
    padding: 20px;
    border-radius: 5px;
}

img {
    max-width: 300px;
    border: 2px solid #333;
}
```

### MIME Types Configuration
```nginx
# Method 1: Include predefined types
http {
    include mime.types;
}

# Method 2: Manual definition
http {
    types {
        text/html html htm;
        text/css css;
        image/png png;
        application/javascript js;
        image/jpeg jpg jpeg;
    }
}
```

## Location Blocks Deep Dive

### Location Block Types and Priority
```nginx
# 1. Exact Match (=)
location = /exact {    # Matches only /exact, not /exact/
}

# 2. Preferential Prefix Match (^~)
location ^~ /images/ {    # Preferred match for /images/
}

# 3. Regular Expression Case-Sensitive (~)
location ~ \.php$ {    # Matches .php files
}

# 4. Regular Expression Case-Insensitive (~*)
location ~* \.(jpg|jpeg|png|gif)$ {    # Matches image files
}

# 5. Prefix Match (none)
location /downloads/ {    # Standard prefix match
}
```

### Practical Location Block Examples
```nginx
# Serving static files
location /static/ {
    root /home/sanskar/mysite;
    autoindex on;    # Enable directory listing
}

# Using alias for different path
location /carbs {
    alias /home/sanskar/mysite/fruits;
    try_files $uri $uri/ =404;
}

# Complex try_files example
location /vegetables {
    root /home/sanskar/mysite;
    try_files /vegetables/veggies.html
              /vegetables/index.html
              /index.html
              =404;
}

# Pattern matching with regular expressions
location ~* /count/[0-9] {
    root /home/sanskar/mysite;
    try_files /index.html =404;
}
```

## Advanced Features

### URL Rewrites and Redirects
```nginx
# Simple redirect
location /crops {
    return 307 /fruits;
}

# Permanent redirect
location /old-page {
    return 301 /new-page;
}

# Complex rewrite
location / {
    rewrite ^/number/(\w+) /count/$1 last;
    rewrite ^/old-url/(.*)$ /new-url/$1 permanent;
}

# Conditional rewrite
if ($host = 'example.com') {
    rewrite ^/(.*)$ http://www.example.com/$1 permanent;
}
```

## Load Balancing

### Basic Round-Robin Configuration
```nginx
# Define upstream servers
upstream backendserver {
    server 127.0.0.1:1111;
    server 127.0.0.1:2222;
    server 127.0.0.1:3333;
    server 127.0.0.1:4444;
}

# Server configuration
server {
    listen 8080;
    
    location / {
        proxy_pass http://backendserver/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Advanced Load Balancing Methods
```nginx
upstream backendserver {
    # Least connections
    least_conn;
    
    # With server weights
    server 127.0.0.1:1111 weight=3;
    server 127.0.0.1:2222 weight=2;
    server 127.0.0.1:3333;  # default weight=1
    
    # With backup server
    server 127.0.0.1:4444 backup;
    
    # Health checks
    check interval=3000 rise=2 fall=5 timeout=1000 type=http;
    check_http_send "HEAD / HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx;
}
```

### Testing Script
```bash
#!/bin/bash
echo "Testing nginx round-robin load balancing..."
echo "Press Ctrl+C to stop"
echo "-----------------------------------"

# Create log file with timestamp
log_file="loadbalancer_test_$(date +%Y%m%d_%H%M%S).log"
echo "Starting test at $(date)" > "$log_file"

# Counter for requests
count=1

while true; do
    echo -n "Request $count: "
    curl -s http://localhost:8080/ | tee -a "$log_file"
    echo " - $(date)" >> "$log_file"
    echo "----------------------------------------"
    count=$((count + 1))
    sleep 1
done
```

## Real-World Examples

### Node.js Application Setup
```javascript
// index.js
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.get('/', (req, res) => {
    res.send(`I am a endpoint running on port ${port}`);
});

app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});
```

### Docker Configuration
```dockerfile
# Dockerfile
FROM node:14-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

EXPOSE 3000
CMD ["node", "index.js"]
```

### Docker Commands
```bash
# Build image
docker build -t node-app .

# Run multiple containers
docker run -d -p 1111:3000 -e PORT=3000 --name server1 node-app
docker run -d -p 2222:3000 -e PORT=3000 --name server2 node-app
docker run -d -p 3333:3000 -e PORT=3000 --name server3 node-app
docker run -d -p 4444:3000 -e PORT=3000 --name server4 node-app
```

## Best Practices

### Security
```nginx
# Hide Nginx version
server_tokens off;

# Add security headers
add_header X-Frame-Options "SAMEORIGIN";
add_header X-XSS-Protection "1; mode=block";
add_header X-Content-Type-Options "nosniff";

# SSL configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers ECDH+AESGCM:ECDH+AES256:ECDH+AES128:DH+3DES:!ADH:!AECDH:!MD5;
```

### Performance
```nginx
# Enable compression
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

# Browser caching
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    add_header Cache-Control "public, no-transform";
}

# Buffer size optimization
client_body_buffer_size 10K;
client_header_buffer_size 1k;
client_max_body_size 8m;
large_client_header_buffers 2 1k;
```

## Troubleshooting

### Common Issues and Solutions

#### Permission Denied Errors
```bash
# Fix permissions
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 755 /var/www/html
```

#### Config Test Failed
```bash
# Check syntax
sudo nginx -t

# Check error logs
sudo tail -f /var/log/nginx/error.log
```

#### 502 Bad Gateway
```nginx
# Increase proxy timeouts
location / {
    proxy_connect_timeout 60s;
    proxy_send_timeout 60s;
    proxy_read_timeout 60s;
    proxy_pass http://backendserver/;
}
```

### Monitoring Commands
```bash
# Monitor access logs
tail -f /var/log/nginx/access.log

# Check error logs
tail -f /var/log/nginx/error.log

# Check open ports
netstat -tulpn | grep nginx

# Check Nginx processes
ps aux | grep nginx
```

### Pro Tips
* Always use `nginx -t` before reloading configuration
* Use CMD+SHIFT+R for hard reload in browser when testing changes
* Keep backup of working configurations
* Use meaningful server and location block names
* Comment your configurations
* Regularly monitor logs for issues
* Keep Nginx updated for security patches
