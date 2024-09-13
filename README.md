# Nginx
web server
reverse proxy server 
load balencer 

# Nginx vs. Apache:

Performance: Nginx excels in handling high-traffic websites and static content efficiently with fewer resources.
Architecture: Nginx is event-driven (non-blocking) vs. Apache’s process/thread-based (blocking) model.


# install
sudo apt-get install nginx
sudo systemctl status nginx


Test Nginx in your browser: Visit http://localhost or your server's IP in a browser, and you should see the Nginx default welcome page.

main conf file :
/etc/nginx/nginx.conf 


#  Serving Static Content
Nginx is very efficient at serving static content like HTML, CSS, and JS files.

1. Setting the Root Directory: In the configuration file (/etc/nginx/sites-available/default):

server {
    listen 80;
    server_name localhost;

    location / {
        root /var/www/html;
        index index.html index.htm;
    }
}
2. Create a Basic HTML File: Place an HTML file in /var/www/html/
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to Nginx</title>
</head>
<body>
    <h1>Hello, Nginx!</h1>
</body>
</html>


3. restaring nginx 
   Now, visiting http://localhost should display the "Hello, Nginx!" page.


#  Reverse Proxy Setup
A reverse proxy forwards client requests to backend servers. For example, Nginx can accept requests and forward them to a Node.js server running on a different port.

1. Setting Up Nginx as a Reverse Proxy
Example Nginx Configuration (for a Node.js server running on port 3000)

server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

2. reload nginx
   sudo systemctl reload nginx


# Load Balancing with Nginx

Load balancing distributes incoming traffic across multiple servers to improve reliability and performance.
Configuring Nginx for Round-Robin Load Balancing:
Example configuration to load balance between two backend servers:


http {
    upstream backend_servers {
        server server1.example.com;
        server server2.example.com;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://backend_servers;
        }
    }
}
upstream: Defines a group of backend servers.
proxy_pass: Forwards requests to the upstream server group.


# SSL/TLS with Nginx

1. Install Certbot:
sudo apt install certbot python3-certbot-nginx

2. sudo certbot --nginx -d yourdomain.com


# Nginx Performance Optimization
1. caching with Nginx:
Nginx allows caching of responses to reduce load on backend servers.


location / {
    proxy_cache my_cache;
    proxy_cache_valid 200 1h;
    proxy_pass http://backend_servers;
}

2. Compression with Gzip:
Enabling Gzip compression reduces file sizes for faster loading times.


gzip on;
gzip_types text/plain application/json text/css application/javascript;

3. Rate Limiting:
To protect your server from DDoS attacks, rate limiting is essential.


limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
location / {
    limit_req zone=one burst=5;
}

# Security with Nginx
1. Blocking Bots:
You can block bad bots by checking the User-Agent:

if ($http_user_agent ~* (BadBot|OtherBadBot)) {
    return 403;
}


2. Basic Authentication:
Add password protection to directories.

Install Apache’s htpasswd tool:

sudo apt install apache2-utils
Create a password file:

sudo htpasswd -c /etc/nginx/.htpasswd username
Update Nginx configuration:

location /secure/ {
    auth_basic "Restricted Content";
    auth_basic_user_file /etc/nginx/.htpasswd;
}



3. Logging and Monitoring
Access Logs and Error Logs:
Logs help you monitor your Nginx server’s performance and troubleshoot errors.


access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
