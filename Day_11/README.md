## Install and Configure Tomcat Server

#### Tasks

a. Install tomcat server on App Server 1

b. Configure it to run on port 6300.

c. There is a ROOT.war file on Jump host at location /tmp.

Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e curl http://stapp01:6300

---

Task a; Install Tomcat on App Server 1

```
# Connect from jump host(bastion)
ssh tony@stapp01

# Install Java (Tomcat dependency)
sudo yum install -y java-11-openjdk

# Install Tomcat
sudo yum install -y tomcat tomcat-webapps tomcat-admin-webapps

```

Enable and start Tomcat service:

```
sudo systemctl enable tomcat
sudo systemctl start tomcat
sudo systemctl status tomcat

```

#### Step 2: Configure Tomcat to Run on Port 6300

```
sudo vi /etc/tomcat/server.xml
```

Find this section (default port 8080):

```
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Change it to 6300:

```
<Connector port="6300" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Restart Tomcat:

```
sudo systemctl restart tomcat
```

#### Step 3: Transfer and Deploy the ROOT.war

From `Jump Host`, copy the `ROOT.war` file to App Server 1:
exit the app server 1 and back into the Jump Host server(bastion)

```
scp /tmp/ROOT.war tony@stapp01:/tmp/
```

On App Server 1, move the WAR file to Tomcat’s webapps directory:

```
sudo mv /tmp/ROOT.war /usr/share/tomcat/webapps/ROOT.war
```

Tomcat will automatically extract and deploy the WAR file.

#### Step 4: Verify Deployment

Run:

```
curl http://stapp01:6300
```
