### Issue

Day by day traffic is increasing on one of the websites managed by the Nautilus production support team. Therefore, the team has observed a degradation in website performance. Following discussions about this issue, the team has decided to deploy this application on a high availability stack i.e on Nautilus infra in Stratos DC. They started the migration last month and it is almost done, as only the LBR server configuration is pending. Configure LBR server as per the information given below:

### Tasks;

a. Install nginx on LBR (load balancer) server.

b. Configure load-balancing with the an http context making use of all App Servers. Ensure that you update only the main Nginx configuration file located at /etc/nginx/nginx.conf.

c. Make sure you do not update the apache port that is already defined in the apache configuration on all app servers, also make sure apache service is up and running on all app servers.

d. Once done, you can access the website using StaticApp button on the top bar.

---

### Task: Configure Nginx as a Load Balancer on the LBR Server

#### Step 1: SSH into the Load Balancer Server

```
ssh loki@stlb01
# password: Mischi3f
```

Explanation:

Connects you from the jump host to the load balancer server (stlb01) using ssh.

#### Step 2: Install Nginx on the Load Balancer

```
sudo yum install nginx -y
```

Explanation:

Installs the Nginx package using YUM package manager.

The -y flag automatically answers “yes” to prompts.

#### Step 3: Enable and Start Nginx

```
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

Explanation:

enable: Start Nginx at boot.

start: Start the Nginx service immediately.

status: Confirm that Nginx is running.

#### Step 4: Configure Nginx as a Load Balancer

```
sudo vi /etc/nginx/nginx.conf
```

Inside the http { ... } block, add an upstream block for your app servers and update the default server block to reverse-proxy traffic:

```
http {
    upstream app_servers {
        server 172.16.238.10:3004;
        server 172.16.238.11:3004;
        server 172.16.238.12:3004;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://app_servers;
        }
    }
}
```

Explanation:

upstream app_servers

Defines a load balancer pool called app_servers

Contains the 3 backend Apache HTTP Server servers with their IP and port

listen 80;

Makes Nginx accept HTTP traffic on port 80 (default)

location / { proxy_pass ... }

Proxies any incoming requests to the upstream backend group

Nginx will automatically use round robin load balancing

#### Step 5: Test and Reload Nginx Configuration

```
sudo nginx -t
sudo systemctl restart nginx
```

Explanation:

- nginx -t tests the configuration syntax for errors.

- systemctl restart nginx restarts the Nginx service to apply changes.

If there are no syntax errors, you’ll see:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

#### Step 6: Verify that is Running on All App Servers

From the jump host, SSH into each app server and check the service:

```
ssh tony@stapp01   # password: Ir0nM@n
ssh steve@stapp02  # password: Am3ric@
ssh banner@stapp03 # password: BigGr33n

sudo systemctl status httpd
```

Explanation:

`systemctl status httpd` confirms that (httpd) is active (running).

This is required so the load balancer can forward requests to them.

#### Step 7: Test the Load Balancer from the Jump Host

```
curl -I http://172.16.238.14
```

Explanation:

curl -I sends a HEAD request to the LBR (load balancer) server.

The load balancer will forward the request to one of the backend app servers.

Expected Output:

```
HTTP/1.1 200 OK
Server: nginx/1.20.1
...
```

You may also see the page content if you do:

```
curl http://172.16.238.14
```

Expected Content:

```
Welcome to xFusionCorp Industries!
```

Result:
The load balancer is now successfully configured on the LBR server to distribute traffic among all 3 app servers running . This resolves the performance issue by ensuring high availability and better load distribution.
