### Apache Multi-Site Configuration on Custom Port (Static Websites Setup)

#### 📘 Objective

xFusionCorp Industries plans to host two static websites on their infrastructure in the Stratos Datacenter.
The development is ongoing, but we want to prepare the web server environment.

This setup ensures:

Apache is installed and running on port 5001.

Two websites (official and apps) are hosted and accessible via:

http://localhost:5001/official/

http://localhost:5001/apps/

#### ⚙️ Steps Performed

##### Step 1: Install Apache (httpd)

Log into App Server 1 (stapp01) as user tony:

```
sudo yum install httpd -y
```

Verify installation:

```
httpd -v
```

Expected output shows Apache version (e.g., Server version: Apache/2.4.x).

##### Step 2: Change Apache Port to 5001

By default, Apache serves on port 80.
We need to change it to 5001.

Edit the main Apache configuration file:

```
sudo vi /etc/httpd/conf/httpd.conf
```

Find the line:

```
Listen 80
```

Change it to:

```
Listen 5001
```

##### Step 3: Copy the Website Backups

On the Jump Host, there are two folders:

```
/home/thor/official
/home/thor/apps
```

Copy them to stapp01 (App Server 1):

```
scp -r /home/thor/official tony@stapp01:/tmp/
scp -r /home/thor/apps tony@stapp01:/tmp/
```

Then log into stapp01 and move them into Apache’s web root:

```
sudo mv /tmp/official /var/www/html/
sudo mv /tmp/apps /var/www/html/
```

Verify:

```
ls /var/www/html/
```

Expected output:

```
official  apps
```

##### Step 4: Configure Apache VirtualHost for Both Sites

Instead of modifying the default Apache config, it’s best practice to create a separate configuration file.

Create a new file:

```
sudo vi /etc/httpd/conf.d/websites.conf
```

Add the following configuration:

```
<VirtualHost *:5001>
    ServerAdmin admin@localhost

    Alias /official /var/www/html/official
    <Directory /var/www/html/official>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    Alias /apps /var/www/html/apps
    <Directory /var/www/html/apps>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/websites_error.log
    CustomLog /var/log/httpd/websites_access.log combined
</VirtualHost>
```

##### Step 5: Start & Enable Apache Service

```
sudo systemctl restart httpd
sudo systemctl enable httpd
sudo systemctl status httpd
```

Confirm it’s running and listening on port 5001:

```
sudo netstat -tulnp | grep httpd
```

Expected:

```
tcp6   0   0 :::5001   :::*   LISTEN   <pid>/httpd
```

##### Step 6: Test the Configuration

Use curl to test both URLs locally on the app server:

```
curl http://localhost:5001/official/
curl http://localhost:5001/apps/
```

Expected Output:

- The first command displays the homepage of the Official Website.

- The second displays the homepage of the Apps Website.

### Final Verification

Run:

```
curl http://localhost:5001/official/
curl http://localhost:5001/apps/
```

You should get valid HTML responses for both.

## 🎉 Task Completed Successfully!
