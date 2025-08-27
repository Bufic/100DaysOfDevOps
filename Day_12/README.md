# Issue:

Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port 5004 (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue.

# Task:

Use tools like telnet, netstat, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings.

Once fixed, you can test the same using command curl http://stapp01:5004 command from jump host.

---

#### Steps to fix issue:

1. Ssh'ed into the stapp01 server and ran the `sudo systemctl status httpd` command to check if the Apache server was active/running. Then i saw it has failed.

```
[tony@stapp01 ~]$ sudo systemctl status httpd
[sudo] password for tony:
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor
 preset: disabled)
   Active: failed (Result: exit-code) since Thu 2025-08-21 10:
53:18 UTC; 3min 30s ago
     Docs: man:httpd.service(8)
  Process: 1076 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (
code=exited, status=1/FAILURE)
 Main PID: 1076 (code=exited, status=1/FAILURE)
   Status: "Reading configuration..."

Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com httpd[1076]: (98)Address
already in use: AH00072: make_sock: could not bind to address 0.0.0.0:500
4
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com httpd[1076]: no listening
 sockets available, shutting down
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com httpd[1076]: AH00015: Una
ble to open logs
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com systemd[1]:
httpd.service: Child 1076 belongs to httpd.service.
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com systemd[1]:
httpd.service: Main process exited, code=exited, status=1/FAIL
URE
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com systemd[1]:
httpd.service: Failed with result 'exit-code'.
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com systemd[1]:
httpd.service: Changed start -> failed
```

Then i tried to run these commands `sudo systemctl start httpd &
sudo systemctl enable httpd` to try and start and enable it on start up. Then i got a control process exit with error code;

```
[tony@stapp01 ~]$ sudo systemctl start httpd
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xe" for details.
[tony@stapp01 ~]$
```

So from the previous log error i got from running `sudo systemctl status httpd`

```
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com httpd[1076]: (98)Address
already in use: AH00072: make_sock: could not bind to address 0.0.0.0:500
4
Aug 21 10:53:18 stapp01.stratos.xfusioncorp.com httpd[1076]: no listening
 sockets available, shutting down
```

This tells me that something else is already listening on port `6400`, not Apache itself.

Next thing for me to do was to find what was listening on port `6400`.
So i ran `sudo netstat -tulnp | grep :6400`, and saw that Sendmail is already bound to port 6400 (on 127.0.0.1).

```
[tony@stapp01 ~]$ sudo netstat -tulnp | grep :6400

tcp        0      0 127.0.0.1:6400          0.0.0.0:*               LISTEN      451/sendmail: accep
[tony@stapp01 ~]$
```

So now i have two choices;
Option 1: Change Apache to another port
Option 2: Stop Sendmail from using port 6400

Seeing as the task was to make sure that Apache runs on port 6400; Sendmail has to be disabled. So i ran `sudo systemctl stop sendmail & sudo systemctl disable sendmail`

```
[tony@stapp01 ~]$ sudo systemctl status sendmail
● sendmail.service - Sendmail Mail Transport Agent
   Loaded: loaded (/usr/lib/systemd/system/sendmail.service; disabled; ve
ndor preset: disabled)
   Active: inactive (dead)

Aug 27 15:16:04 stapp01.stratos.xfusioncorp.com systemd[1]:
sendmail.service: Main process exited, code=exited, status=0/S
UCCESS
Aug 27 15:16:04 stapp01.stratos.xfusioncorp.com systemd[1]: sendmail.serv
ice: Succeeded.
```

Then i restarted the Httpd service

```
[tony@stapp01 ~]$ sudo systemctl start httpd
[tony@stapp01 ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendo
r preset: disabled)
   Active: active (running) since Wed 2025-08-27 15:22:07 UTC;
 36s ago
     Docs: man:httpd.service(8)
 Main PID: 1551 (httpd)
   Status: "Running, listening on: port 6400"
    Tasks: 213 (limit: 411434)
   Memory: 22.9M
   CGroup: /docker/c4dc997c7cf5050d3f62349190ad6b1829df41836c4aeb08641e10
13a450deaf/system.slice/httpd.service
           ├─1551 /usr/sbin/httpd -DFOREGROUND
           ├─1576 /usr/sbin/httpd -DFOREGROUND
           ├─1577 /usr/sbin/httpd -DFOREGROUND
           ├─1578 /usr/sbin/httpd -DFOREGROUND
           └─1579 /usr/sbin/httpd -DFOREGROUND
```

And Voila the Apache service is running smoothly on port `6400`.

```
[tony@stapp01 ~]$ sudo netstat -tulnp | grep :6400
tcp        0      0 0.0.0.0:6400            0.0.0.0:*               LISTEN      1551/httpd
[tony@stapp01 ~]$
```

---

So the next step is to make sure that we can access the Apache service from the Jumphost(bastion). But first lets test the apache service by running `curl http://localhost:6400`. And Voila

```
    /*]]>*/
  </style>
</head>
<body>
  <header class="container">
    <div class="row header">
      <div class="header-title">HTTP Server <strong>Test Page</strong></div>
    </div>
  </header>
  <main class="container">
    <div class="row">
      <div class="col">
        <p class="lead">This page is used to test the proper operation of the HTTP server after it has been installed. If you can read this page it means that this site is working properly. This server is powered by <a href="http://centos.org">CentOS</a>.</p>
      </div>
```

So lets exit the stapp01 server where the Apache server is running and test from the jumphost.

So from the jumphost i ran `curl http://stapp01:6400` and got

```
thor@jumphost ~$ curl http://stapp01:6400
curl: (7) Failed to connect to stapp01 port 6400: No route to host
thor@jumphost ~$
```

Which means it’s not Apache anymore — it’s a network reachability or firewall issue.

So let me check on the apache server if Apache is listening on all interfaces or if it's only bound to localhost.

So i ran `sudo netstat -tulnp | grep` and got;

```
[tony@stapp01 ~]$ sudo netstat -tulnp | grep :6400
tcp        0      0 0.0.0.0:6400            0.0.0.0:*               LISTEN      1551/httpd
```

This shows Apache is bound to 0.0.0.0, meaning it is listening on all interfaces (not just 127.0.0.1). So the problem was not Apache itself.

At this point, the issue had to be network/firewall related.

###### Step: Check firewall / iptables

firewalld was not running, and SELinux was disabled, so I checked iptables rules. By default, iptables may be blocking inbound traffic on non-standard ports like 6400.

So I added a rule to allow inbound TCP traffic on port 6400: `sudo iptables -I INPUT -p tcp --dport 6400 -j ACCEPT`

```
[tony@stapp01 ~]$ sudo iptables -I INPUT -p tcp --dport 6400 -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -L -n | grep 6400
# Warning: iptables-legacy tables present, use iptables-legacy to see them
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:6400
[tony@stapp01 ~]$
```

Then i tried accessing the apache service from the Jumphost again by running `curl http://stapp01:6400`

And then i was able to reach the service

```
  </style>
</head>
<body>
  <header class="container">
    <div class="row header">
      <div class="header-title">HTTP Server <strong>Test Page</strong></div>
    </div>
  </header>
  <main class="container">
    <div class="row">
      <div class="col">
        <p class="lead">This page is used to test the proper operation of the HTTP server after it has been installed. If you can read this page it means that this site is working properly. This server is powered by <a href="http://centos.org">CentOS</a>.</p>
      </div>
    </div>
    <hr>
    <div class="row">
      <div class="col-xl-6">
        <h2>If you are a member of the general public:</h2>
        <p>The website you just visited is either experiencing problems or is undergoing routine maintenance.</p>
        <p>If you would like to let the administrators of this website know that you've seen this page instead of the page you expected, you should send them e-mail. In general, mail sent to the name "webmaster" and directed to the website's domain should reach the appropriate person.</p>
        <p>For example, if you experienced problems while visiting www.example.com, you should send e-mail to "webmaster@example.com".</p>
      </div>
      <div class="col-xl-6">
        <h2>If you are the website administrator:</h2>
        <p>You may now add content to the webroot directory. Note that until you do so, people visiting your website will see this page, and not your content.</p>
        <p>For systems using the Apache HTTP Server: You may now add content to the directory <code>/var/www/html/</code>. Note that until you do so, people visiting your website will see this page, and not your content. To prevent this page from ever being used, follow the instructions in the file <code>/etc/httpd/conf.d/welcome.conf</code>.</p>
        <p>For systems using NGINX: You should now put your content in a location of your choice and edit the <code>root</code> configuration directive in the <strong>nginx</strong> configuration file <code>/etc/nginx/nginx.conf</code>.</p>
        <p><a href="https://www.centos.org/"><img src="/icons/poweredby.png" alt="[ Powered by CentOS ]"></a> <img src="poweredby.png" alt="[ Powered by CentOS ]"></p>
      </div>
    </div>
    <hr>
    <div class="row">
      <div class="col">
        <h2 class="alert-heading">Important note!</h2>
        <p>The CentOS Project has nothing to do with this website or its content, it just provides the software that makes the website run.</p>
        <p>If you have issues with the content of this site, contact the owner of the domain, not the CentOS project. Unless you intended to visit CentOS.org, the CentOS Project does not have anything to do with this website, the content or the lack of it.</p>
        <p>For example, if this website is www.example.com, you would find the owner of the example.com domain at the following WHOIS server: <a href="http://www.internic.net/whois.html">http://www.internic.net/whois.html</a></p>
      </div>
    </div>
  </main>
  <footer class="container">
    <div>&#xA9; 2021 The CentOS Project | <a href="https://www.centos.org/legal/">Legal</a> | <a href="https://www.centos.org/legal/privacy/">Privacy</a></div>
  </footer>
</body>
</html>
thor@jumphost ~$
```
