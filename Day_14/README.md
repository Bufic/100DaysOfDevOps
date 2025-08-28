## Linux Process Troubleshooting.

---

### Issue:

The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.

### Task:

Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not have hosted any code yet on these servers, so you don’t need to worry if Apache isn’t serving any pages. Just make sure the service is up and running. Also, make sure Apache is running on port 3000 on all app servers.

---

#### Steps to fix:

##### Step 1: Identify the faulty app host

We have 3 app servers in Stratos DC:

stapp01 → 172.16.238.10 (tony / Ir0nM@n)

stapp02 → 172.16.238.11 (steve / Am3ric@)

stapp03 → 172.16.238.12 (banner / BigGr33n)

We need to log into each server (via the jump host) and check Apache’s status.

Example; repeat for all servers.

```
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03

sudo systemctl status httpd
```

After checking all servers, only stapp01 → 172.16.238.10 (tony / Ir0nM@n) has a failed status of Apache. The others have an Active apache status and are running on port 3000.

I verified the port by running `sudo ss -tulnp | grep httpd` on all servers.

```
[steve@stapp02 ~]$ sudo ss -tulnp | grep httpd
tcp   LISTEN 0      511          0.0.0.0:3000       0.0.0.0:*    users:(("httpd",pid=1682,fd=3),("httpd",pid=1681,fd=3),("httpd",pid=1680,fd=3),("httpd",pid=1672,fd=3))
[steve@stapp02 ~]$
```

```
[banner@stapp03 ~]$ sudo systemctl status httpd

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for banner:
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; pr>
     Active: active (running) since Thu 2025-08-28 08:42:42 UTC; 5min ago
       Docs: man:httpd.service(8)
   Main PID: 1665 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0>
      Tasks: 177 (limit: 411434)
     Memory: 15.5M
     CGroup: /docker/864869b78205a73d41b551623288dd23bbfd8125369a099d699>
             ├─1665 /usr/sbin/httpd -DFOREGROUND
             ├─1672 /usr/sbin/httpd -DFOREGROUND
             ├─1673 /usr/sbin/httpd -DFOREGROUND
             ├─1674 /usr/sbin/httpd -DFOREGROUND
             └─1675 /usr/sbin/httpd -DFOREGROUND

Aug 28 08:46:22 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:46:32 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:46:42 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:46:52 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:47:02 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:47:12 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:47:22 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:47:32 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:47:42 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>
Aug 28 08:47:52 stapp03.stratos.xfusioncorp.com systemd[1]: httpd.servic>

[banner@stapp03 ~]$ sudo ss -tulnp | grep httpd
tcp   LISTEN 0      511          0.0.0.0:3000       0.0.0.0:*    users:(("httpd",pid=1675,fd=3),("httpd",pid=1674,fd=3),("httpd",pid=1673,fd=3),("httpd",pid=1665,fd=3))
[banner@stapp03 ~]$
```

##### Step 2: Fix the faulty host stapp01

```
thor@jumphost ~$ ssh tony@stapp01
The authenticity of host 'stapp01 (172.16.238.10)' can't be established.
ED25519 key fingerprint is SHA256:yHfHGhqXCBD4DoZFPnUf9hRwq0W6xDOIQFAWdsrsnEI.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'stapp01' (ED25519) to the list of known hosts.
tony@stapp01's password:
[tony@stapp01 ~]$ sudo systemctl status httpd

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Thu 2025-08-28 08:42:42 UTC; 1min 23s ago
     Docs: man:httpd(8)
           man:apachectl(8)
  Process: 798 ExecStop=/bin/kill -WINCH ${MAINPID} (code=exited, status=1/FAILURE)
  Process: 797 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
 Main PID: 797 (code=exited, status=1/FAILURE)

Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com httpd[797]: AH00558: h...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com httpd[797]: (98)Addres...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com httpd[797]: no listeni...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com httpd[797]: AH00015: U...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.serv...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com kill[798]: kill: canno...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.serv...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com systemd[1]: Failed to ...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com systemd[1]: Unit httpd...
Aug 28 08:42:42 stapp01.stratos.xfusioncorp.com systemd[1]: httpd.serv...
Hint: Some lines were ellipsized, use -l to show in full.
[tony@stapp01 ~]$
```

What the error says:

From your logs:

```
(98)Address already in use: AH00072: make_sock: could not bind to address [::]:80
no listening sockets available, shutting down
```

This means:

Something else is already using port 3000 on stapp01, so Apache can’t bind to it.

##### How to fix on stapp01

Confirm what’s using port 3000 by running `sudo ss -tulnp | grep 3000`

```
tcp    LISTEN     0      10     127.0.0.1:3000                  *:*                   users:(("sendmail",pid=772,fd=4))
```

sendmail is already listening on port 3000, so Apache (httpd) can’t bind to it. That’s why it fails.

To fix, we have to free up port 3000 for Apache by disabling Sendmail.

```
sudo systemctl stop sendmail
sudo systemctl disable sendmail
```

And then we restart the Apache service

```
sudo systemctl start httpd
sudo systemctl status httpd
```

```
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2025-08-28 09:10:49 UTC; 10s ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 1132 (httpd)
   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /docker/2457d78a0f0da46215870548ea4fd85a62e9ecd42c9bda134c81548cc0baa2e4/system.slice/httpd.service
           ├─1132 /usr/sbin/httpd -DFOREGROUND
           ├─1133 /usr/sbin/httpd -DFOREGROUND
           ├─1134 /usr/sbin/httpd -DFOREGROUND
           ├─1135 /usr/sbin/httpd -DFOREGROUND
           ├─1136 /usr/sbin/httpd -DFOREGROUND
           └─1137 /usr/sbin/httpd -DFOREGROUND

Aug 28 09:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: Starting T...
Aug 28 09:10:49 stapp01.stratos.xfusioncorp.com httpd[1132]: AH00558: ...
Aug 28 09:10:49 stapp01.stratos.xfusioncorp.com systemd[1]: Started Th...
Hint: Some lines were ellipsized, use -l to show in full.


[tony@stapp01 ~]$ sudo ss -tulnp | grep httpd
tcp    LISTEN     0      511       *:3000                  *:*                   users:(("httpd",pid=1137,fd=3),("httpd",pid=1136,fd=3),("httpd",pid=1135,fd=3),("httpd",pid=1134,fd=3),("httpd",pid=1133,fd=3),("httpd",pid=1132,fd=3))
[tony@stapp01 ~]$
```
