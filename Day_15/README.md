### Issue

The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 3 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below:

### Tasks

1. Install and configure nginx on App Server 3.
2. On App Server 3 there is a self signed SSL certificate and key present at location /tmp/nautilus.crt and /tmp/nautilus.key. Move them to some appropriate location and deploy the same in Nginx.
3. Create an index.html file with content Welcome! under Nginx document root.
4. For final testing try to access the App Server 3 link (either hostname or IP) from jump host using curl command. For example curl -Ik https://<app-server-ip>/.

---

### Infrastructure Details;

```
Infrastructure Details

Server: stapp03 (App Server 3)

IP: 172.16.238.12

User: banner

Password: BigGr33n
```

#### Step 1: Install Nginx on App Server 3

SSH into App Server 3 from the Jump Host:

```
ssh banner@stapp03
```

Install Nginx:

```
sudo yum install -y nginx
```

Enable and start the Nginx service:

```
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

```
Complete!
[banner@stapp03 ~]$ sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Wed 2025-09-10 11:20:36 UTC; 131ms ago
    Process: 2648 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 2649 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 2656 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 2663 (nginx)
      Tasks: 17 (limit: 411434)
     Memory: 18.1M
     CGroup: /docker/8141ab3dd7bc75dcd701f50c1b66bf6ab6f534c0b657efd32b40d7d3e18cb351/system.slice/nginx.service
             ├─2663 "nginx: master process /usr/sbin/nginx"
             ├─2664 "nginx: worker process"
             ├─2665 "nginx: worker process"
             ├─2666 "nginx: worker process"
             ├─2667 "nginx: worker process"
             ├─2668 "nginx: worker process"
             ├─2669 "nginx: worker process"
```

At this point, Nginx should be running on port 80.

#### Step 2: Configure SSL Certificates for Nginx

Move the provided certificates to a secure location:

```
sudo mkdir -p /etc/nginx/ssl
sudo mv /tmp/nautilus.crt /etc/nginx/ssl/
sudo mv /tmp/nautilus.key /etc/nginx/ssl/
sudo chmod 600 /etc/nginx/ssl/nautilus.*
```

Update the Nginx configuration to enable HTTPS. Create a new config file:

```
sudo vi /etc/nginx/conf.d/ssl.conf
```

Add the following:

```
server {
    listen 443 ssl;
    server_name stapp03.stratos.xfusioncorp.com;

    ssl_certificate     /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key /etc/nginx/ssl/nautilus.key;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

Test Nginx configuration:

```
sudo nginx -t
```

You should see;

```
[banner@stapp03 ~]$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Restart Nginx to apply changes:

```
sudo systemctl restart nginx
```

#### Step 3: Create index.html

Inside the Nginx document root:

```
echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
```

#### Step 4: Final Testing from Jump Host

```
exit
```

Test HTTPS access:

```
curl -Ik https://172.16.238.12/
```

You should see an HTTP response like:

```
[banner@stapp03 ~]$ exit
logout
Connection to stapp03 closed.
thor@jumphost ~$ curl -Ik https://172.16.238.12/
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Wed, 10 Sep 2025 11:43:18 GMT
Content-Type: text/html
Content-Length: 9
Last-Modified: Wed, 10 Sep 2025 11:41:07 GMT
Connection: keep-alive
ETag: "68c163d3-9"
Accept-Ranges: bytes

thor@jumphost ~$
```
