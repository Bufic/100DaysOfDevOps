xFusionCorp Industries is planning to host a WordPress website on their infra in Stratos Datacenter. They have already done infrastructure configuration—for example, on the storage server they already have a shared directory /vaw/www/html that is mounted on each app host under /var/www/html directory. Please perform the following steps to accomplish the task:

a. Install httpd, php and its dependencies on all app hosts.

b. Apache should serve on port 6000 within the apps.

c. Install/Configure MariaDB server on DB Server.

d. Create a database named kodekloud_db4 and create a database user named kodekloud_pop identified as password BruCStnMT5. Further make sure this newly created user is able to perform all operation on the database you created.

e. Finally you should be able to access the website on LBR link, by clicking on the App button on the top bar. You should see a message like App is able to connect to the database using user kodekloud_pop

##### Objective

Perform the following steps to configure and deploy the environment:

1. Install Apache (httpd), PHP, and dependencies on all app servers.

2. Configure Apache to serve on port 8089.

3. Install and configure MariaDB on the database server.

4. Create a database (kodekloud_db4) and user (kodekloud_sam) with the required privileges.

5. Verify that the website is accessible via the Load Balancer (LBR) and displays:

`App is able to connect to the database using user kodekloud_sam`

#### Step 1: SSH into App Server 1

```
ssh tony@stapp01
# password: Ir0nM@n
```

#### Step 2: Install Apache, PHP, and Dependencies

```
sudo yum install httpd php php-mysqlnd php-fpm -y
```

Explanation:

httpd: Installs Apache web server

php and php-mysqlnd: Installs PHP and MySQL module to connect PHP with MariaDB

php-fpm: FastCGI process manager (for performance)

#### Step 3: Configure Apache to Listen on Port 8089

Edit the Apache configuration file:

```
sudo vi /etc/httpd/conf/httpd.conf
```

Find this line:

```
Listen: 80
```

Change it to:

```
Listen: 8089
```

Save and exit (:wq).

#### Step 4: Enable and Start Apache Service

```
sudo systemctl enable httpd
sudo systemctl start httpd
sudo systemctl status httpd
```

##### Expected Output:

Apache should be running and listening on port 8089.

Verify the port:

```
sudo ss -tuln | grep 8089
```

#### Step 6: Create Website File in Shared Directory

```
echo "App is able to connect to the database using user kodekloud_sam" | sudo tee /var/www/html/index.php
```

##### Explanation:

This replaces the default PHP info page with your custom message, stored in the shared directory.

Since the directory is shared, the same page will appear on stapp02 and stapp03.

#### Step 7: Configure MariaDB on Database Server

SSH into the database server:

```
ssh peter@stdb01
# password: Sp!dy
```

##### Install MariaDB (if not installed)

```
sudo yum install mariadb-server -y
```

#### Step 8: Create Database and User

Login to MariaDB shell:

```
sudo mysql
```

Run the following SQL commands:

```
CREATE DATABASE kodekloud_db4;
CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'B4zNgHA7Ya';
GRANT ALL PRIVILEGES ON kodekloud_db4.* TO 'kodekloud_sam'@'%';
FLUSH PRIVILEGES;
EXIT;
```

##### Explanation:

`CREATE DATABASE`: Creates a new database.

`CREATE USER`: Creates a user with a secure password.

`GRANT ALL PRIVILEGES`: Gives the user full access to that database.

`'%'`: Allows the user to connect from any host (needed for remote app servers).

#### Step 9: Test Database Connection (from any App Server)

On any app server:

```
mysql -h 172.16.239.10 -u kodekloud_sam -p
# password: B4zNgHA7Ya
```

You should successfully connect to MariaDB.

#### Step 10: Verify Website from Load Balancer

From the Jump Host, run:

```
curl -I http://stlb01:80
```

Expected Output:

```
HTTP/1.1 200 OK
Server: Apache/2.4.62 (CentOS Stream)
X-Powered-By: PHP/8.0.30
Content-Type: text/html; charset=UTF-8
```

Then test actual content:

```
curl http://stlb01
```

Expected Response:

```
App is able to connect to the database using user kodekloud_sam
```

#### Final Output

When accessed via Load Balancer (http://stlb01
), the web page displays:

```
App is able to connect to the database using user kodekloud_sam
```

Task completed successfully!
