---
layout: post
title:  "Dev URLs without Pow"
date:   2014-12-27 21:48:00
categories:
---

*TL;DR: I had to do a bunch of research in order to set up pretty URLs to apps running on my localhost at a particular port, so I decided to write up the steps I followed in one place in case anyone else wants to do this. Scroll down past the reasons I wanted to do this for the instructions.*

I recently decided to use a non-admin user with OSX Yosemite on a daily basis, mostly on the recommendation of [a security researcher who discovered a vulnerability in OSX](http://www.macworld.com/article/2841965/swedish-hacker-finds-serious-vulnerability-in-os-x-yosemite.html), and because it sounds like a good idea in general.

The only problem I've encountered so far is in installing [Pow](http://pow.cx/)-- the script to install it wants sudo, but it also wants to be run as the current user. There are instructions on the wiki for [installation as a standard user](https://github.com/basecamp/pow/wiki/Installation-under-Standard-user), but they appear to be out of date; I got errors when trying to use `npm run-script`. I also got errors when trying to install it from source. So ¯\\\_(ツ)\_/¯.

The thing is, I was only using Pow for its pretty URL creation anyway; I use foreman or grunt to run my apps instead of Pow's rack server. So I set out to get URLs like `http://my-project.dev` to serve up a rails app that I would usually access at `http://localhost:3000`.

My first thought was that I could just use `/etc/hosts`, but [that only works for host names](http://stackoverflow.com/questions/10729034/can-i-map-a-hostname-and-a-port-with-etc-hosts), not host names *and* ports. Luckily, Apache can do this and, conveniently enough, is included with OSX!

BUT WAIT! I *do* still need to use `/etc/hosts`, so that the host goes to localhost where Apache will handle it.

## Instructions

So, to spell it all out, I logged in as my admin user (since I'm running from a non-admin day-to-day now) and edited `/etc/hosts` using `sudo`:

```
$ login admin
password:
admin$ sudo vim /etc/hosts
```

In order to add a new entry for `my-project.dev`. So now, my `/etc/hosts` file looks like:

```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
255.255.255.255	broadcasthost
::1             localhost

127.0.0.1       my-project.dev
```

At this point, still as the admin user, I started up apache by doing:

```
admin$ sudo apachectl start
```

And then I could go to `http://my-project.dev` and see the Apache "It works!" page. [We're halfway there!](https://www.youtube.com/watch?v=lDK9QqIzhwk)

Now, edit `/etc/apache2/httpd.conf`, find this part about Virtual Hosts and uncomment the `Include` line so it looks like this:

```
# Virtual hosts
Include /private/etc/apache2/extra/httpd-vhosts.conf
```

Then, edit THAT file, `/private/etc/apache2/extra/httpd-vhosts.conf`, remove the two sample `<VirtualHost>`s, and insert a new one that looks like this, as recommended by [this ServerFault answer](http://serverfault.com/a/195831/174036):

```
NameVirtualHost *:80

<VirtualHost *>
    ServerName my-project.dev
    ProxyPreserveHost On

    # setup the proxy
    <Proxy *>
        Order allow,deny
        Allow from all
    </Proxy>
    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/
</VirtualHost>
```

Then restart apache:

```
admin$ sudo apachectl restart
```

And, if you've got your app running at port 3000, you should be able to visit `http://my-project.dev` in your browser and get to your app!
