---
layout: post
title: "Your Hubot is listening on *"
modified: 2014-10-26 09:35:17 -0700
tags: [hubot, nodejs, security, nmap]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: 
---

This morning I decided to do a little security audit on a server I have that happens to face the outside world. This same server gets hit 5-7 times a day with automated tools attempting to log in through ssh (to only get perma-banned by [fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) :D).

A good first step to see what an attacker might see on your server is to run [nmap](http://nmap.org/). With a typical scan, nmap will show you what ports are open and available to the outside world. Usually, your web server shouldn't have too many open ports. The more ports you have open, the higher the attack surface.

If you have nmap installed, you can check your own server with the command below (__THIS IS IMPORTANT__: you must either own the server or have permission from the owner to do a scan like this):

```
nmap -T4 -A -v yourwebsite.com
```

From the output here, nmap will tell you what ports are open and what service is likely running on those ports. It will also attempt to determine the operating system, but that is less reliable.

I was expecting to only see port 22 and port 80 (ssh and web), however something interesting popped up...port 8080. At first that worried me, because nmap reported it as an http proxy, which is something I have not set up at all. I poured over my different configs related to nginx or anything web related and didn't see anything. I even stopped nginx and redid the scan and it was still there! Strange....I finally looked at what was listening on my machine with netstat:

{% highlight javascript %}
brian@macha:~ $ netstat -anp | grep -i 8080
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:8080              0.0.0.0:*                   LISTEN      2128/node
{% endhighlight %}

Sure enough, something was listening on 8080, but it wasn't nginx, it was Node! Well, on this machine that's Hubot. And it turns out, if you don't specify a binding address, Hubot decides to make itself open to the world.

The code is here on [hubot/src/robot.coffee](https://github.com/github/hubot/blob/master/src/robot.coffee#L287):

{% highlight coffeescript %}
try
  @server = app.listen(process.env.PORT || 8080, process.env.BIND_ADDRESS || '0.0.0.0')
  @router = app
catch err
  @logger.error "Error trying to start HTTP server: #{err}\n#{err.stack}"
  process.exit(1)
{% endhighlight %}

What does this mean? Well by default, Hubot will be open to the outside world. _Does it really matter?_ Maybe, maybe not. But I don't have a reason for it to, so there's no reason for it to be open to the world. According to the code snippate, if we set a BIND_ADDRESS environment variable, Hubot will bind to that instead of the default `0.0.0.0`. My deploy script looks something like this:


{% highlight bash %}
#! /bin/bash

HUBOT_IRC_SERVER=irc.freenode.net \
HUBOT_IRC_ROOMS="#mychannel" \
HUBOT_IRC_NICK="MyHubotNick" \
HUBOT_IRC_UNFLOOD="true" \
PORT=1234 \
BIND_ADDRESS=localhost \
bin/hubot -a irc --name MyHubotNick
{% endhighlight %}

Setting BIND_ADDRESS will make the Hubot framework bind to that instead of `0.0.0.0`. Doing another nmap scan will show that there is nothing on 8080 anymore! Also checking netstat, we can see that Hubot is now listening on localhost (127.0.0.1), instead of *.


{% highlight bash %}
brian@macha:~ $ netstat -anp | grep -i 1234
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:1234              0.0.0.0:*                   LISTEN      2128/node
{% endhighlight %}
