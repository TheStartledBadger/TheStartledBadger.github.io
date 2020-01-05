---
title: Running Jekyll in WSL
date:   2020-01-05
layout: post
comments: true
---

So far I've been writing blog posts using a virtual machine (VirtualBox) running linux. It's quite a pain.  Every time I want to use it
I start up the VM and immediately have to apply a bunch of updates.  Well, I don't *have to* install the updates, but psychologically,
I *do* have to!

Anyhow, Windows 10 has WSL (Windows Subsystem for Linux) which seems like it should be good enough to run jekyll.  There are quite a few
pages out there already on this subject but I tried following various sets of instructions and didn't have much success last time I 
looked in to it.

Today I decided to try again and, joy of joys, it seems I've got something working.  Moreover, it wasn't all that hard.

If you search for help no this subject the top result is a [page from the jekyll site itself](https://jekyllrb.com/docs/installation/windows/), which seems quite sensible really. Sadly, this didn't work for me for a couple of reasons
* `gem install` didn't want to work unless I used sudo (no permissions on directories)
* If I ran `gem install` as root, it still failed.  `libz` and `gdbm` at a minimum wouldn't compile

I was close to giving up when I stumbled on another page which gave a different approach using `rvm`. [This page](https://gorails.com/setup/windows/10) did the trick for me...

Briefly, the steps required after installing WSL (which I already had set up running Ubuntu 18.04) were:

```
sudo apt-get update

sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev

sudo apt-get install libgdbm-dev libncurses5-dev automake libtool bison libffi-dev

gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB

curl -sSL https://get.rvm.io | bash -s stable

source ~/.rvm/scripts/rvm

rvm install 2.7.0

rvm use 2.7.0 --default

ruby -v
```

Then, because I was wanting to use this to work with `github.io` pages, a couple of extra things:
```
gem install bundler
gem install jekyll
gem install github-pages
```

Then I cloned my existing blog site and successfully wrote this article locally using
```
jekyll serve
```

from the root of the blog directory structure to view this page as I was writing it. Magic.
