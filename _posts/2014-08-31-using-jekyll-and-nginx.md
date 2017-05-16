---
layout: post
title: "Using Jekyll and Nginx"
date: 2014-08-31 22:00:00 -0700
tags:
- jekyll
- ruby
- nginx
- web
---

Thanks to the ruby gem [Jekyll](http://jekyllrb.com/), it's now easier than ever to have a simple dynamic website with just a few commands and a little bit of ruby and markdown.

## Setting up your development environment

To get started, if you haven't already, I recommend installing [rvm](http://rvm.io/). RVM makes it easy to move in between multiple versions of ruby, and also makes managing different gems between projects a little bit easier. As their website suggests, you can get rvm with a simple command in bash:

```
\curl -sSL https://get.rvm.io | bash -s stable
```

Once rvm installs all of the required packages, you need to pick what version of ruby you are interested in developing in. For the purposes of this blog, we'll just pick Ruby 2.0.0:

{% highlight bash %}
rvm install 2.0.0
{% endhighlight %}

So far, so good. Only a few commands. The next step will be installing the `jekyll` ruby gem so we can start developing our website. This should be the same as the steps detailed on the [main Jekyll website](http://jekyllrb.com/). If you want, you can use rvm's gemset functionality, but it isn't required. Go ahead and install Jekyll, and then generate a new website with it after installation:

{% highlight bash %}
gem install jekyll
jekyll new my-website
{% endhighlight %}

Doing so should give you a new Jekyll website with this folder layout structure below. If you want to dig more into each folder, I recommend reading the [Jekyll Directory structure docs](http://jekyllrb.com/docs/structure/) that go into more detail. For now, here's the basic structure that Jekyll gives you.

{% highlight bash %}
.
├── _config.yml
├── _includes
│   ├── footer.html
│   ├── head.html
│   └── header.html
├── _layouts
│   ├── default.html
│   ├── page.html
│   └── post.html
├── _posts
│   └── 2014-08-31-welcome-to-jekyll.markdown
├── about.md
├── css
│   └── main.css
├── feed.xml
└── index.html

4 directories, 12 files
{% endhighlight %}

Testing the changes you make to your new website locally with Jekyll is extremely easy. Once you're ready to see what your website looks like in a browser, type the following command within the directory of your project:

{% highlight bash %}
jekyll serve -w
{% endhighlight %}

The `-w` flag will automatically reload if Jekyll notices any files have been updated from an editor. This is extremely useful if you want to test a lot of changes and don't want to constantly tear down and bring up Jekyll every time you want to see the change.

If you did everything right, you should see this website on [localhost:4000](http://localhost:4000)!

<figure>
  <img src="/images/jekyll-new.png" alt="new website">
  <figcaption>A blank jekyll page</figcaption>
</figure>

From here, there are a lot of different choices you could make. It's your website, so that part is up to you! If you are interested in Jekyll themes, there are plenty out there to choose from. This site uses one! If you want a more blog oriented experience, the ruby gem [Octopress](http://octopress.org/) has a lot of great features and themes.

## Setting up your web server

Now that you're all done designing your website and adding lots of content, it's time to show it off to the world! Let's log into your remote machine through ssh and get started....

### Setting up Nginx

The first thing you should do on your remote web server is install Nginx. This step depends on what package manager you have. Let's just assume you're using yum. First you need to install __EPEL__ or "Extra Packages for Enterprise Linux". Then we can install nginx after we add those packages to yum.

{% highlight bash %}
sudo su -c 'rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm'
sudo yum install nginx
{% endhighlight %}

When nginx first installs, it does not automatically start after installation. At least...that is the behavior on CentOS. This is good, since we might want to make a few changes to the nginx.conf file before setting up our website.

The default config file _should_ be ok, but if you'd like to change something all of the Nginx configuration files are located in `/etc/nginx`. I recommend checking out `/etc/nginx/nginx.conf` even if you don't plan on changing anything. For more information, feel free to read the [Nginx beginners guide](http://nginx.org/en/docs/beginners_guide.html).

### Deploying Jekyll

Now that Nginx is installed and configured, we need to use Jekyll to build everything up for us. This can be done by using the simple build command in the root of your working Jekyll projects directory:

{% highlight bash %}
jekyll build
{% endhighlight %}

That command will generate your site into the _site folder. How you get this folder over to your web server is up to you. Whether it be SFTP, SCP, or cloning your website over with git and building it on your web server, as long as you place the resulting built website code in the folder Nginx is serving websites out of, you will be good. In the default case, that folder is `/usr/share/nginx/html`. Placing your website in there will allow nginx to serve out your Jekyll website. Once that is done, starting up Nginx is the last step!


{% highlight bash %}
service nginx start
{% endhighlight %}

If everything worked, you should have a brand new website!
