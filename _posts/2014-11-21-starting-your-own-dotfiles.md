---
layout: post
title: "Starting Your Own Dotfiles"
date: 2014-11-21 17:30:00 -0800
headerImage: false
tag:
- dotfiles
- configuration
- automation
- nix
star: false
category: blog
projects: true
author: briancain
---

## What the heck are dotfiles, and why should you care?

If you're familiar with Unix/Linux operating systems you may have opened up a terminal and looked at all of the various files within your home directory. You may have even viewed and edited a few of them, like a `.bashrc` or `.vimrc`. Here's an example of what your home directory might look like:

{% highlight bash %}
[vagrant@centos ~]$ ls -lah
total 44K
drwx------. 6 vagrant vagrant 4.0K Nov 20 18:10 .
drwxr-xr-x. 3 root    root    4.0K Feb 18  2014 ..
-rw-r--r--. 1 vagrant vagrant   18 Jul 18  2013 .bash_logout
-rw-r--r--. 1 vagrant vagrant  176 Jul 18  2013 .bash_profile
lrwxrwxrwx  1 vagrant vagrant   30 Nov 20 18:09 .bashrc -> /home/vagrant/.dotfiles/bashrc
drwxr-xr-x  4 vagrant vagrant 4.0K Nov 20 18:10 .dotfiles
-rw-rw-r--  1 vagrant vagrant  138 Nov 20 18:09 .gitconfig
drwxrw----  3 vagrant vagrant 4.0K Nov 20 18:00 .pki
drwx------  2 vagrant vagrant 4.0K Feb 18  2014 .ssh
-rw-r--r--  1 vagrant vagrant    5 Feb 18  2014 .vbox_version
drwxr-xr-x  3 vagrant vagrant 4.0K Nov 20 18:09 .vim
-rw-------  1 vagrant vagrant  652 Nov 20 18:10 .viminfo
lrwxrwxrwx  1 vagrant vagrant   29 Nov 20 18:09 .vimrc -> /home/vagrant/.dotfiles/vimrc
{% endhighlight %}

So what exactly are these "dot" files?

To start, the dot or '.' in front of the file means that it is actually a hidden file on the system. So to actually see these files, you either have to enable the ability to view those hidden files (if you are using a gui file viewer) or you have to use a specific flag (`-a`) on the command line with `ls`.

Frequently, applications like vim, bash, zsh, irssi, etc use these hidden files or directories as a way to store user specific configurations.

So why should you care?

Having a dotfiles project provides a quick way to stand up your operating system to configure it exactly what you want. Have any aliases that you can't live without? Or perhaps you're picky on how your vim behaves? Dotfiles also provide a way for you to sync and backup specific aspects of the tools you use across all of your physical and virtual machines.

## Getting Started

[briancain/starter-dotfiles](https://github.com/briancain/starter-dotfiles)

In the tradition of dotfiles, I have shared a starter repo that I encourage you to fork onto your own Github profile. This starter repo is just the bare minimum. It simply installs a few basic tools depending on your OS, and then sets up the few "dot" files like a bashrc, vimrc, etc that you might want and expand upon. To set them up, follow the simple instructions below:

Clone my dotfiles into your home directory under the name `.dotfiles`

    $ git clone https://github.com/briancain/starter-dotfiles.git ~/.dotfiles

Then `cd ~/.dotfiles` and run my simple installer bash script

    $ ./install.bash

It will ask you a few questions (like your favorite shell), and assume you wish to use vim :)

### Syncing your changes

If you make any changes to your vimrc, bashrc, or zshrc, since those files are symlinked it will also update the repository. This comes in handy when you want to sync those changes to another machine. If you have made changes in your repository, simply `cd ~/.dotfiles` and use git to commit those changes and push it up to your fork on Github.

If you have your own rc file that you would like to add to your repo, you can either manually symlink it or place it in the `~/.dotfiles` directory without the '.' in front of the name. Then if you re-run the installer it will symlink everything for you in your home directory. Placing it in your `~/.dotfiles` directory allows for it to be uploaded to Github and synced across your machines.

## Additional Information

### Shells

- [Oh My ZSH](http://ohmyz.sh/)
  + [Themes](https://github.com/robbyrussell/oh-my-zsh/wiki/Themes)
  + [Plugins](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins)

### Getting started with Vim and Vundle

- [Learn Vim Progressively](http://yannesposito.com/Scratch/en/blog/Learn-Vim-Progressively/)
- [About Vundle, the vim plugin manager](https://github.com/gmarik/Vundle.vim#about)

### Other Resources

- [My personal dotfiles](https://github.com/briancain/dotfiles)
- [Github does dotfiles](http://dotfiles.github.io/)
