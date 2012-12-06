---
layout: post
title: "Munin Setup for  Nginx and Mongodb Monitoring on Ubuntu 11.10"
date: 2012-12-06 22:00
comments: true
categories: [Technology, Programming, English, Work]
---

I was trying to set up some kind of monitoring system for our soon-to-be-live
backend system. After some quick research, I decided to use give Munin a try
since it has pretty good community support for all kinds of plugins. 
Their [homepage](munin-monitoring.org/) plus couple google searches help me
to present the following walkthrough.

### Installation

Installation is pretty easy on Ubuntu server, for the munin master:

```
apt-get install munin
```

for the munin node:

```
apt-get install munin-node munin-plugins-extra
```

### Configuration
#### munin master

First you need configure `/etc/munin/munin.conf`, in general you just need to
specify the host tree, bascially all munin nodes you want to monitor. Something
like this:

```
[server1.domain.com]
    address [your_server_ip_address]
```

One caveat here is to make sure the server name `server1.domain.com` you specify
here matches the configuration later you do on your node. If these two do not
match, it will not get data at all.

Then you need to make sure your munin monitoring webpage can be accessed. If you
are using nginx like me, just create a file under
`/etc/nginx/sites-available/munin` with config like this:

```
server {
  listen 80;
  server_name monitor.domain.com;
  location / {
    # Host Based auth
    #allow 11.12.22.55/32;
    #deny all;
    # Passwd auth
    auth_basic            "Restricted";
    auth_basic_user_file  munin_auth_pass;
    root /var/www/munin;
  }
 }
```

I am using basic password authentication and for that you need
to create a password file. For the above configuration, you need to have a file
`/etc/ngnix/munin_auth_pass`:

```
username:encrypted_password
```

If you use ruby, the easiest thing to get the encrypted_password would be:

```
& irb
1.9.3 :001> "password".crpyt('salt')
=> "sa3tHJ3/KuYvI"
```

Now you are all set to have a munin monitoring page running and password
proteced at `monitor.domain.com`.

#### munin node

Munin node, by default, is using port 4949, you can configure it and just make sure the
port is not blocked by the firewall, which happened to me in the
beginning. Configure `/etc/munin/munin-node.conf` and two key configurations are:

```
host_name server1.domain.com

allow ^127\.0\.0\.1$
allow [regex_of_ip_address_of_munin_master]
```

You need to make sure the host name you set here matches the one you put in your
munin master conf. Now if you do a `sudo service munin-node restart`, you should
be able to see graphs on your munin master for several existing monitor items
like system, networking and etc. 

If you want to monitor more items like nginx and mongodb, you can find most
third party plugins in [munin github project](https://github.com/munin-monitoring/contrib/tree/master/plugins/). Let's say you want to monitor nginx, what you need to do is:

```
cd /usr/share/munin/plugins
sudo wget -O nginx_combined https://raw.github.com/munin-monitoring/contrib/master/plugins/nginx/nginx-combined
sudo chmod +x nginx_combined
sudo ln -s /usr/share/munin/plugins/nginx_combined /etc/munin/plugins/nginx_combined_[munin node ip address]
sudo service munin-node restart
```

And you need to configure `/etc/munin/plugin-conf.d/munin-node` to add these two
lines:

```
[nginx*]
env.url http://localhost/nginx_status
```

You can add more plugins with similar approach and different plugins may have
their own custom config to make sure it gets data correctly. 

This whole process took me more than a whole afternoon to figure out, hopefully
this blog can help you move faster with munin. 

Happy monitoring!

### References
* [http://play.datalude.com/blog/2012/01/munin-nginx-mysql-ubuntu/](http://play.datalude.com/blog/2012/01/munin-nginx-mysql-ubuntu/)
* [http://www.cowboycoded.com/2011/01/12/password-protect-your-entire-site-with-nginx/](http://www.cowboycoded.com/2011/01/12/password-protect-your-entire-site-with-nginx/)
