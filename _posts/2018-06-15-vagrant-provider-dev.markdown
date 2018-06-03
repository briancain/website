---
title: "Vagrant Provider Development Explained: A quickstart guide"
layout: post
date: 2016-11-29 18:30:00 -0800
headerImage: false
tag:
- cli
- virtual
- tool
- api
- ruby
- vagrant
- development
- provider
star: false
category: blog
projects: true
author: briancain
description: Description here
---

I found myself in a dilemma a few months ago: I had written a command line tool
called [vmfloaty](https://github.com/briancain/vmfloaty) for
[vmpooler](https://github.com/puppetlabs/vmpooler), and that helped reduce the
amount of time I spent worrying about how to get virtual machines to do my job
at [Puppet](https://puppet.com/). But I still had an issue with provisioning and
managing the state of those virtual machines in a timely manner.

Being a software engineer at Puppet, you could say that I’m someone who loves automation. However even after automating the process of grabbing machines from vmpooler, I was still left with the problem of setting up those machines for my development environment. I could probably just use SSH in a `for` loop, but I had already heavily relied on Vagrant in the past to do this setup for me. I didn’t really want to go back to using existing Vagrant providers with VMware or Virtualbox because I always ended up hitting the limitations of my own laptop hardware.

Pushing off the virtual machines to vmpooler sure was convenient, but once they were up and running it look a lot of effort to manually set them all up to develop with. I needed something like Vagrant to help me provision those machines and manage their lifecycle. Being that vmpooler is a internally home-grown project at Puppet, there are no existing Vagrant providers out there for it. This left me with a good opportunity to fill the need and hopefully make my job a little easier.

So how does one go about writing their own Vagrant provider? There are already a lot of good providers out there, so surely there’s some sort of guide, right? Sure! There **is** a pretty [basic starting guide](https://www.vagrantup.com/docs/plugins/providers.html) from HashiCorp, giving you a high level of what’s possible. But I still felt like I needed more guidance before getting started. After searching around for more detailed guides, I was unfortunately left with the option of staring at existing providers and attempting to learn how to write one by reading source code.

I hope to just cover a couple of basic-but-important components that are required in a Vagrant provider. This isn’t meant to be an extensive “how-to” guide that will show you how to write a provider from start to finish, but hopefully it can save you some trouble when writing your provider in Vagrant and understand what goes into making a Vagrant provider work.

If you’re curious about what you’re in for, or want to jump to a specific section, here’s what I’m planning on covering:

__Table of contents__

- [Provider Configuration](#provider-configuration)
  + Every provider requires some sort of configuration. This will step you through what that means and where to start.
- [Provider Actions](#provider-actions)
  + Actions help make your provider function. This will give you a basic example of one of those actions.
- [I18n and Log Messages](#i18n-and-log-messages)
  + Internationalizing your log messages is an important part of writing a provider, and this gives you some basic examples on how to do so
- [Vagrant Box](#vagrant-box)
  + What the heck is a Vagrant box and why should my provider care? Hopefully this will clear up the confusion

#### Special Thanks

Thank you [Ryan McKern](https://twitter.com/the_mckern) for proof reading and editing!

## Provider Configuration

If your provider requires extra configuration (it almost certainly will), you need to provide a configuration class that defines all of the config variables that a user might need to set within a Vagrantfile. Take this basic code sample from a Vagrantfile:

{% highlight ruby %}
config.vm.provider :vmpooler do |vmpooler|
  vmpooler.url = "https://vmpooler.com/api/v1"
  vmpooler.os = "centos-7-x86_64"
  vmpooler.ttl = 24
  vmpooler.password = "secretpassword"
end
{% endhighlight %}

All of these settings within the `:vmpooler` block are config settings defined by my config class. When a user defines these settings, they are stored within Vagrants internal memory and can be retrieved later (see the later post for more information about actions). Each config setting is defined as an `attr_accessor` (which is just a fancy Ruby way of saying the variable is both a getter and a setter). It’s also a good idea to write some documentation above the variable explaining what it’s for and what type your provider is expecting it to be. Based on the Vagrant providers out there, most authors format their config documentation using [Yard](http://yardoc.org). For example, here is how the `token` setting is defined in my provider:

{% highlight ruby %}
# The token used to obtain vms
#
# @return [String]
attr_accessor :token
{% endhighlight %}

Your config class should inherit from Vagrant.plugin("2", :config) and contain at least two instance methods: `#initialize` and `#finalize!`.

The `#initialize` method is where you will set up all of the config settings that your provider will need. Unlike many other Ruby projects, Vagrant expects your config settings in initialize to be set to `UNSET_VALUE`. `UNSET_VALUE` is Vagrant’s way of initializing a config value without setting it to `nil`, in case the user explicitly wants that variable to be `nil`. In Vagrant, `UNSET_VALUE` is just a shortcut for setting your Vagrant config setting to `Object.new`. By using this common Vagrant convention, a provider plugin doesn’t have to do any guesswork or sanity-checking to figure out the user's intention on what the config setting was set to. `UNSET_VALUE` is really a fancy way of saying “I want this config variable defined as a new object, but don’t actually use it right now”.

Your `#finalize!` method is always the last method called by Vagrant and is used to do any last minute variable configuration or validation. It also is where you should handle any `UNSET_VALUE` config variables or do any validation in case the user misconfigured a setting within their Vagrantfile. You should set any `UNSET_VALUE` config variables to `nil` if any still are `UNSET_VALUE` at this point. Otherwise that may lead to some fun exceptions later when your provider attempts to use these variables as-is. For example: If your provider expects your `token` variable to be a string but you never finalize it to nil and it wasn’t set by a Vagrantfile, when Vagrant attempts to use it as such later you will likely get an exception, since that token defaults to a new Ruby object.

Now that you have some basic config settings ready to go, you should be able to access them in your provider actions by calling  `env[:machine].provider_config`. Read more about actions in the next section!

## Provider Actions

Actions are the heart of a Vagrant provider: they’re what makes it functional. These are the functions that help your provider interact and manage virtual machine life cycles, configuration states, etc. They’re also the interface between the Vagrant CLI and the backend that provides the virtual machines themselves.

### Action Controller

This internal middleware uses the Vagrant Builder class to define various Vagrant actions for the provider. Each method in your actions controller can combine multiple actions to complete a single task. The syntax can be a little confusing and complex at first since it uses Ruby’s block iteration, but a simple example without too much branching logic really can boil it down to what is happening:

{% highlight ruby %}
def self.action_prepare_boot
  Vagrant::Action::Builder.new.tap do |b|
    b.use Provision
    b.use SyncFolders
    b.use SetHostname
  end
end
{% endhighlight %}

In this step, we’re simply calling three Vagrant actions on the virtual machine. Each action will be executed in the order specified. So in this case we’ll call `Provision`, `SyncFolders`, and finally `SetHostname` in that order on the machine.

### Actions

Actions are discrete functions used by the action controller class to perform various tasks against your virtual machine. But first, let’s talk about what an action looks like.

All actions inherit from the [Action](https://github.com/mitchellh/vagrant/blob/master/lib/vagrant/action.rb) module from Vagrant. They have a couple key methods required:

**#initialize**

As the name suggested, this method is what’s called to initialize your action. Most if not all of my `#initialize` methods end up looking as simple as this:

{% highlight ruby %}
def initialize(app, env)
  @app = app
  @logger = Log4r::Logger.new("vagrant_vmpooler::action::create_server")
end
{% endhighlight %}

**#call**

This is the method invoked when the action is called from the actions controller. It takes an `env` parameter which is a pretty messy (and huge) data structure Vagrant uses to store state about everything on your machine related to Vagrant. You shouldn’t have to care about what’s in there for the most part except for a few keys. Within there, you can usually find a `:machine` key, which points to a data structure that should give you access to the provider config, the machine’s id, and so on.

`#call` is where you can put all of your business logic related to the action. Finally, it’s important to put call back to the action middleware in your actions call method so Vagrant can execute any other outstanding actions afterwards:

{% highlight ruby %}
def call(env)
  #
  # other code goes here
  #
  @app.call(env)
end
{% endhighlight %}

### Action Example

Let’s take this simple example of:

{% highlight bash %}
$ vagrant destroy
{% endhighlight %}

The `destroy` in `vagrant destroy` is an action defined in the `Action` controller class.

{% highlight ruby %}
# This goes at the top of your Action controller
include Vagrant::Action::Builtin

# ...
# This action is called when `vagrant destroy` is executed.
# example source https://github.com/briancain/vagrant-vmpooler/blob/master/lib/vagrant-vmpooler/action.rb#L11
def self.action_destroy
  Vagrant::Action::Builder.new.tap do |b|
    b.use ConfigValidate
    b.use Call, DestroyConfirm do |env, b1|
      if env[:result]
        b1.use DeleteServer
      else
        b1.use MessageWillNotDestroy
      end
    end
  end
end
{% endhighlight %}

First, the action validates the provider config defined in the Vagrantfile, queries the user to confirm that they want to delete the server, and if so calls our own custom action `DeleteServer`. Otherwise it calls an action that prints a message that it won’t delete the server.

There are some actions that you do not need to write yourself (like `ConfigValidate`). By including `include Vagrant::Action::Builtin` in your action controller, you are able to use these built-in actions. There isn’t a complete list of what actions you get by doing this from what I could find, so I ended up having to [read the source code](https://github.com/mitchellh/vagrant/tree/master/lib/vagrant/action/builtin) to see what was available.

Now let’s get into the action `DeleteServer`:

{% highlight ruby %}
# full source here: https://github.com/briancain/vagrant-vmpooler/blob/master/lib/vagrant-vmpooler/action/delete_server.rb
def call(env)
  machine = env[:machine]
  id = machine.id

  if id
    # various settings required by my provider
    provider_config = machine.provider_config

    token = provider_config.token
    url = provider_config.url
    verbose = provider_config.verbose

    env[:ui].info(I18n.t("vagrant_vmpooler.deleting_server"))

    # api call to delete server with ID
     os = []
     os.push(id)
     response_body = Pooler.delete(verbose, url, os, token)

    if response_body[id]['ok'] == false
      # something went wrong
      env[:ui].warn(I18n.t("vagrant_vmpooler.not_deleted"))
    else
      # server marked for deletion
      env[:ui].info(I18n.t("vagrant_vmpooler.deleted"))
      # setting the machine id to nil here removes
      # it from Vagrants internal state
      env[:machine].id = nil
    end
  else
    # if no id is found, machine doesn't exist
    env[:ui].info(I18n.t("vagrant_vmpooler.not_created"))
  end

  # Go back to my actions controller and execute any outstanding actions
  @app.call(env)
end
{% endhighlight %}

The `delete_server` action reads in the provider config to acquire the basic settings needed to talk to its backend, and then makes the API request to delete the given server based on the ID. It then has some handling to ensure things went alright, and that’s it!

Now that we’ve written an action, we’re ready to load it in the controller class:

{% highlight ruby %}
# somewhere below your methods in your Action controller
...
action_root = Pathname.new(File.expand_path("../action", __FILE__))
autoload :DeleteServer, action_root.join("delete_server")
# More actions get autoloaded here
{% endhighlight %}

One they’re loaded, they are referred to by the autoload name. For example, the action `delete_server` will now be referred to as `DeleteServer` in the action controller class.

If you’d like a more complicated example of an action, check out my plugins [CreateServer](https://github.com/briancain/vagrant-vmpooler/blob/master/lib/vagrant-vmpooler/action/create_server.rb) action.

## I18n and Log Messages

Most if not all of the vagrant providers I’ve read use localization. It’s generally a good idea to bake in localization or `i18n` to your project. If you’re looking to have your provider work in languages other than English this will help make the process a lot easier. The locale file itself is a `yaml` file made up of a hierarchy of keys and log messages. The top level key should correspond to the locale you’re writing for (like `en` for `English`). Otherwise the structure isn’t too important, but pick something that makes sense so that they’re easy to reference while developing. A simple example could be you want a log message that tells the user a server is being destroyed. In your locale file, you might have a structure that looks like this for English or French:

{% highlight yaml %}
# locales/en.yml
en:
  vagrant_vmpooler:
    destroy_server:
      Destroying server…
{% endhighlight %}

{% highlight yaml %}
# locales/fr.yml
fr:
  vagrant_vmpooler:
    destroy_server:
      Détruire le serveur...
{% endhighlight %}

Then simply when you want to use that message, you can reference it like so:

{% highlight ruby %}
env[:ui].info(I18n.t("vagrant_vmpooler.destroy_server"))
{% endhighlight %}

And Vagrant will be able to use the appropriate log message based on the user's system locale on their computer.

## Vagrant box

When writing a Vagrantfile, each defined virtual machine block needs to set a `box` setting so that Vagrant can use that as a base image when creating a virtual machine. In some Vagrant provider cases, there is no actual `box` (like AWS or OpenStack) so the box ends up just being a dummy box that vagrant can use to get information for the given provider to use.

So what the heck is a Vagrant box then if it isn’t a virtual machine image? In my case, it ends up being a compressed archive of your `metadata.json` file. In other words it’s a piece of data that gives Vagrant more information about the virtualization software that it’s going to use. Here’s what my `metadata.json` file looks like for my provider:

{% highlight json %}
{ "provider": "vmpooler" }
{% endhighlight %}

In this case, all my provider needs to know when a user specifies this box to use in their Vagrantfile, Vagrant should use the `vmpooler` provider.

Once you’ve created a metadata.json file, you can compress that into a box file for Vagrant to read like so:

{% highlight bash %}
tar -czvf dummy.box metadata.json
vagrant box add dummy.box --name dummy
{% endhighlight %}

Now when you want to use your custom provider with the box you created, you can simply add it by specifying this in your Vagrantfile:

{% highlight ruby %}
Vagrant.configure("2") do |config|
  config.vm.box = "dummy"
  # other stuff goes here
end
{% endhighlight %}

Where “dummy” is the name of the box you gave it when you added it with `vagrant box add`.

## My provider

So what does my provider look like? If you’re like to see how I did, please check out my repository here:

- [vagrant-vmpooler](https://github.com/briancain/vagrant-vmpooler)

## Helpful Repositories and Resources

- [vagrant-aws](https://github.com/mitchellh/vagrant-aws)
- [vagrant-openstack](https://github.com/ggiamarchi/vagrant-openstack-provider)
- [HashiCorp Plugin Development: Providers](https://www.vagrantup.com/docs/plugins/providers.html)
