## PHP Application Deployment on stapp03 (Stratos DC)

##### 1. Overview

This document describes how to deploy the PHP-based Nautilus application on App Server 3 (stapp03).

The deployment ensures:

- nginx as the web server listening on port 8094

- PHP-FPM 8.1 using a UNIX socket

- Correct integration between nginx and PHP-FPM

- Application files (index.php and info.php) remain unmodified

- Validator-friendly setup for Nautilus automated checks

##### 2. Prerequisites

- Access to stapp03 via SSH

- Root or sudo privileges

- Jump host access for remote testing

##### 3. Deployment Steps

STEP 1 — Login to App Server 3

```
ssh banner@stapp03.stratos.xfusioncorp.com
```

STEP 2 — Remove Existing PHP (Avoid Conflicts)

```
sudo yum remove -y php php-fpm php-cli php-common
```

Verify removal:

```
php -v
```

Expected outcome: bash: php: command not found

STEP 3 — Enable Repositories

```
sudo yum install -y epel-release
sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

Expected outcome: EPEL and Remi repositories installed for EL9.

STEP 4 — Reset PHP Modules

```
sudo yum module reset php -y
```

Expected outcome:

```
Resetting modules:
 php
Complete!
```

STEP 5 — Enable PHP 8.1 Module

```
sudo yum module enable php:remi-8.1 -y
```

Verify:

```
yum module list php
```

Expected outcome:

```
 php remi-8.1 [e] (enabled)
```

STEP 6 — Install PHP-FPM 8.1

```
sudo yum install -y php php-fpm php-cli php-common php-mbstring php-xml php-mysqlnd
```

Verify PHP-FPM version:

```
php-fpm -v
```

Expected outcome:

```
PHP 8.1.x (fpm-fcgi)
```

STEP 7 — Enable and Start PHP-FPM

```
sudo systemctl enable php-fpm
sudo systemctl start php-fpm
```

Verify:

```
systemctl status php-fpm
```

Expected outcome: Active: active (running)

STEP 8 — Configure PHP-FPM UNIX Socket

```
sudo mkdir -p /var/run/php-fpm
sudo chown nginx:nginx /var/run/php-fpm
sudo chmod 755 /var/run/php-fpm
```

Edit PHP-FPM pool:

```
sudo vi /etc/php-fpm.d/www.conf
```

Ensure these lines:

```
user = nginx
group = nginx

listen = /var/run/php-fpm/default.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

Restart PHP-FPM:

```
sudo systemctl restart php-fpm
```

Verify socket:

```
ls -l /var/run/php-fpm/default.sock
```

Expected outcome:

```
srw-rw---- nginx nginx /var/run/php-fpm/default.sock
```

STEP 9 — Install and Start nginx

```
sudo yum install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

STEP 10 — Configure nginx for PHP
Create a config file:

```
sudo vi /etc/nginx/conf.d/php_app.conf
```

Add:

```
server {
    listen 8094;
    server_name stapp03;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

Test configuration:

```
sudo nginx -t
```

Expected outcome:

```
syntax is ok
test is successful
```

Restart nginx:

```
sudo systemctl restart nginx
```

STEP 11 — Verify Application Files

```
ls -l /var/www/html/index.php
ls -l /var/www/html/info.php
```

Expected outcome: Files exist and are untouched.

STEP 12 — Test from Jump Host

```
curl http://stapp03:8094/index.php
```

Expected outcome: Welcome to xFusionCorp Industries!

The PHP application is now fully deployed on stapp03 and accessible via port 8094.
The setup is validator-safe, secure, and follows all Nautilus infrastructure requirements.
