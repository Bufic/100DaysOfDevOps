# IPtables Installation And Configuration.

---

#### Issue;

We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. Our security team has raised a concern that right now Apache’s port i.e 3000 is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:

#### Tasks;

1. Install iptables and all its dependencies on each app host.

2. Block incoming port 3000 on all apps for everyone except for LBR host.

3. Make sure the rules remain, even after system reboot.

---

#### Step 1:

SSH into each app server (stapp01, stapp02, stapp03) from the jump host and install iptables

```
sudo yum install -y iptables iptables-services
```

```
[tony@stapp01 ~]$ sudo yum install -y iptables iptables-services

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for tony:
Last metadata expiration check: 0:05:12 ago on Wed Aug 27 20:00:42 2025.
Package iptables-legacy-1.8.10-2.2.el9.x86_64 is already installed.
Dependencies resolved.
=========================================================================
 Package                 Arch      Version               Repo       Size
=========================================================================
Installing:
 iptables-services       noarch    1.8.10-11.1.el9       epel       17 k
Upgrading:
 iptables-legacy         x86_64    1.8.10-11.1.el9       epel       50 k
 iptables-legacy-libs    x86_64    1.8.10-11.1.el9       epel       38 k
 iptables-libs           x86_64    1.8.10-11.el9         baseos    462 k

Transaction Summary
=========================================================================
Install  1 Package
Upgrade  3 Packages

Total download size: 567 k
Downloading Packages:
(1/4): iptables-services-1.8.10-11.1.el9  78 kB/s |  17 kB     00:00
(2/4): iptables-legacy-1.8.10-11.1.el9.x 204 kB/s |  50 kB     00:00
(3/4): iptables-legacy-libs-1.8.10-11.1. 487 kB/s |  38 kB     00:00
(4/4): iptables-libs-1.8.10-11.el9.x86_6 1.3 MB/s | 462 kB     00:00
```

#### Step 2:

Configure iptables Rules

We want:

1. Allow incoming port 3000 only from the LBR (172.16.238.14)

2. Deny everyone else

3. Leave other essential traffic (like SSH/loopback) untouched

On each app server:

```
sudo iptables -F     # flush existing rules (optional if clean system)
sudo iptables -A INPUT -p tcp -s 172.16.238.14 --dport 3000 -j ACCEPT   # allow from LBR
sudo iptables -A INPUT -p tcp --dport 3000 -j DROP                      # block all others
```

Verify rules:
By running `sudo iptables -L -n --line-numbers`

You should see something like:

```
[tony@stapp01 ~]$ sudo iptables -F
[tony@stapp01 ~]$ sudo iptables -A INPUT -p tcp -s 172.16.238.14 --dport 3000 -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -A INPUT -p tcp --dport 3000 -j DROP
[tony@stapp01 ~]$ sudo iptables -L -n --line-numbers
Chain INPUT (policy ACCEPT)
num  target     prot opt source               destination
1    ACCEPT     tcp  --  172.16.238.14        0.0.0.0/0            tcp dpt:3000
2    DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3000

Chain FORWARD (policy ACCEPT)
num  target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
num  target     prot opt source               destination

Chain DOCKER (0 references)
num  target     prot opt source               destination

Chain DOCKER-ISOLATION-STAGE-1 (0 references)
num  target     prot opt source               destination

Chain DOCKER-ISOLATION-STAGE-2 (0 references)
num  target     prot opt source               destination

Chain DOCKER-USER (0 references)
num  target     prot opt source               destination
[tony@stapp01 ~]$
```

#### Step 3:

Persist Rules (survive reboot)

By default, iptables rules are not persistent. We need to save them.

```
sudo service iptables save
sudo systemctl enable iptables
sudo systemctl start iptables
```

```
[tony@stapp01 ~]$ sudo service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
[tony@stapp01 ~]$ sudo systemctl enable iptables
Created symlink /etc/systemd/system/multi-user.target.wants/iptables.service → /usr/lib/systemd/system/iptables.service.
[tony@stapp01 ~]$ sudo systemctl start iptables
[tony@stapp01 ~]$
```

Check that persistence is set up by running `sudo iptables -L -n` and `sudo systemctl is-enabled iptables`

```
[tony@stapp01 ~]$ sudo iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.16.238.14        0.0.0.0/0            tcp dpt:3000
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3000
```

```
[tony@stapp01 ~]$ sudo systemctl is-enabled iptables
enabled
[tony@stapp01 ~]$
```

#### Step 4:

Test Access
Exit stapp01 server and ssh into stlb01 the Load Balancer host (stlb01: 172.16.238.14).

From stlb01, run:

```
curl http://stapp01:3000
```

This Should work.

```
  </header>
  <div class="hr">
    <div class="hr__centos-color-0"></div>
    <div class="hr__centos-color-1"></div>
    <div class="hr__centos-color-2"></div>
    <div class="hr__centos-color-3"></div>
  </div>
  <main class="page">
    <article class="page__content">
      <div class="row">
        <div class="col-xl-6">
          <h5>If you are a member of the general public:</h5>
          <p>The website you just visited is either experiencing problems or is undergoing routine maintenance.</p>
          <p>If you would like to let the administrators of this website know that you've seen this page instead of the page you expected, you should send them e-mail. In general, mail sent to the name "webmaster" and directed to the website's domain should reach the appropriate person.</p>
          <p>For example, if you experienced problems while visiting www.example.com, you should send e-mail to "webmaster@example.com".</p>
          <div class="alert alert-info" role="alert">
            <h5 class="alert-heading">Important Note!</h5>
            <p>The CentOS Project has nothing to do with this website or its content, it just provides the software that makes the website run.</p>
            <p>If you have issues with the content of this site, contact the owner of the domain, not the CentOS project. Unless you intended to visit CentOS.org, the CentOS Project does not have anything to do with this website, the content or the lack of it.</p>
            <p>For example, if this website is www.example.com, you would find the owner of the example.com domain at the following WHOIS server: <a class="alert-link" href="http://www.internic.net/whois.html">http://www.internic.net/whois.html</a></p>
          </div>
        </div>
        <div class="col-xl-6">
          <h5>If you are the website administrator:</h5>
          <p>You may now add content to the webroot directory. Note that until you do so, people visiting your website will see this page, and not your content.</p>
          <h6>For systems using the Apache HTTP Server:</h6>
          <p>You may now add content to the directory <code>/var/www/html/</code>. Note that until you do so, people visiting your website will see this page, and not your content. To prevent this page from ever being used, follow the instructions in the file <code>/etc/httpd/conf.d/welcome.conf</code>.</p>
          <p class="small"><a href="https://apache.org">Apache™</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.</p>
          <h6>For systems using NGINX:</h6>
          <p>You should now put your content in a location of your choice and edit the <code>root</code> configuration directive in the <strong>nginx</strong> configuration file <code>/etc/nginx/nginx.conf</code>.</p>
          <p class="small"><a href="https://nginx.com">NGINX™</a> is a registered trademark of <a href="https://www.f5.com">F5 Networks, Inc.</a></p>
          <p><a href="https://www.centos.org/"><img src="/icons/poweredby.png" alt="[ Powered by CentOS ]" /></a> <img src="poweredby.png" alt="[ Powered by CentOS ]" /></p>
          <p class="small"><a href="https://www.centos.org/">CentOS</a> is a registered trademark of <a href="https://www.redhat.com/">Red Hat Inc.</a></p>
        </div>
      </div>
    </article>
  </main>
  <div class="hr">
    <div class="hr__centos-color-0"></div>
    <div class="hr__centos-color-1"></div>
    <div class="hr__centos-color-2"></div>
    <div class="hr__centos-color-3"></div>
  </div>
  <footer class="footer">
    <div class="container">
      <div class="row">
        <section class="links">
          <h6><i class="fas fa-info-circle"></i> About</h6>
          <ul>
            <li>
              <a href="/about">About CentOS</a>
            </li>
            <li>
              <a href="https://wiki.centos.org/FAQ">Frequently Asked Questions (FAQs)</a>
            </li>
            <li>
              <a href="https://wiki.centos.org/SpecialInterestGroups">Special Interest Groups (SIGs)</a>
            </li>
            <li>
              <a href="/variants">CentOS Variants</a>
            </li>
            <li>
              <a href="/about/governance">Governance</a>
            </li>
            <li>
              <a href="/code-of-conduct">Code of Conduct</a>
            </li>
          </ul>
        </section>
        <section class="links">
          <h6><i class="fas fa-users"></i> Community</h6>
          <ul>
            <li>
              <a href="https://wiki.centos.org/Contribute">Contribute</a>
            </li>
            <li>
              <a href="https://www.centos.org/forums/">Forums</a>
            </li>
            <li>
              <a href="https://wiki.centos.org/GettingHelp/ListInfo">Mailing Lists</a>
            </li>
            <li>
              <a href="https://wiki.centos.org/irc">IRC</a>
            </li>
            <li>
              <a href="/community/calendar/">Calendar &amp; IRC Meeting List</a>
            </li>
            <li>
              <a href="http://planet.centos.org/">Planet</a>
            </li>
            <li>
              <a href="https://wiki.centos.org/ReportBugs">Submit Bug</a>
            </li>
          </ul>
        </section>
        <section class="project">
          <h2>The CentOS Project</h2>
          <p class="lead">Community-driven free software effort focused on delivering a robust open source ecosystem around a Linux platform.</p>
          <div class="lead social">
            <a href="https://www.facebook.com/groups/centosproject/"><i class="fab fa-facebook-f"></i></a> <a href="https://twitter.com/CentOS"><i class="fab fa-twitter"></i></a> <a href="https://youtube.com/TheCentOSProject"><i class="fab fa-youtube"></i></a> <a href="https://www.linkedin.com/groups/22405"><i class="fab fa-linkedin"></i></a> <a href="https://www.reddit.com/r/CentOS/"><i class="fab fa-reddit"></i></a>
          </div>
        </section>
      </div>
      <div class="row">
        <section class="copyright">
          <p>Copyright © 2021 The CentOS Project | <a href="/legal">Legal</a> | <a href="/legal/privacy">Privacy</a> | <a href="https://git.centos.org/centos/centos.org">Site source</a></p>
        </section>
      </div>
    </div>
  </footer>
</body>
</html>
[loki@stlb01 ~]$
```

And then from jump_host or any other host:

```
curl http://stapp01:3000
```

This should fail(connection refused/timeout).

```
thor@jumphost ~$ curl http://stapp01:8082

curl: (28) Failed to connect to stapp01 port 8082: Connection timed out

```
