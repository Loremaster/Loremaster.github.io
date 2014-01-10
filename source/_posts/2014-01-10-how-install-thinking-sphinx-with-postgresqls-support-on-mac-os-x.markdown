---
layout: post
title: "How install Thinking Sphinx with PostgreSQL's support on Mac OS X"
date: 2014-01-10 13:05:00 +0400
comments: true
categories:
---

Thinking Sphinx is a library, which allows you use Sphinx - full text search server. But setup can be tricky, if you use PostgreSQL. In this post I will show you you how setup it.

First of all, make sure that you have installed Mysql. The best way to install it is just use homebrew:

{% codeblock Terminal lang:bash %}
$ brew install mysql
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/mysql-5.6.15.mavericks.bottle.tar.gz
######################################################################## 100.0%
==> Pouring mysql-5.6.15.mavericks.bottle.tar.gz
==> Caveats
A "/etc/my.cnf" from another install may interfere with a Homebrew-built
server starting up correctly.

To connect:
    mysql -uroot

To have launchd start mysql at login:
    ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
Then to load mysql now:
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
Or, if you don't want/need launchctl, you can just run:
    mysql.server start
{% endcodeblock %}

I recommend you to copy and past commands from homebrew's output after installation.

If you don't want to use homebrew (but i strongly recommend to **use** homebrew), then you should install x64 MySQL server from [official site](http://dev.mysql.com/downloads/mysql/). Make sure to use x64, you don't want to get any weird errors (I guess that you use at least OS X 10.6).

Now you should install Sphinx. And it's a tricky part. If you would install it without any flag or with just mysql support, you would get such weird errors:

{% codeblock Terminal lang:bash %}
$ rake ts:index
Generating configuration to /Users/serj/Projects/gearup/config/development.sphinx.conf
Sphinx 2.1.3-dev (r4319)
Copyright (c) 2001-2013, Andrew Aksyonoff
Copyright (c) 2008-2013, Sphinx Technologies Inc (http://sphinxsearch.com)

using config file '/Users/serj/Projects/gearup/config/development.sphinx.conf'...
indexing index 'service_core'...
ERROR: source 'service_core_0': unknown type 'pgsql'; skipping.
ERROR: index 'service_core': failed to configure some of the sources, will not index.
total 0 reads, 0.000 sec, 0.0 kb/call avg, 0.0 msec/call avg
total 0 writes, 0.000 sec, 0.0 kb/call avg, 0.0 msec/call avg
{% endcodeblock %}

To avoid them you should instal sphinx with mysql **and** postgres support:

{% codeblock Terminal lang:bash %}
$ brew install sphinx --mysql --pgsql
==> Downloading http://sphinxsearch.com/files/sphinx-2.1.3-release.tar.gz
Already downloaded: /Library/Caches/Homebrew/sphinx-2.1.3.tar.gz
==> Downloading http://snowball.tartarus.org/dist/libstemmer_c.tgz
Already downloaded: /Library/Caches/Homebrew/sphinx--stemmer-c.tgz
==> ./configure --prefix=/usr/local/Cellar/sphinx/2.1.3 --localstatedir=/usr/local/var --with-libstemmer --with-mysql --with-pgsql
==> make install
==> Caveats
Sphinx has been compiled with libstemmer support.

Sphinx depends on either MySQL or PostreSQL as a datasource.

You can install these with Homebrew with:
  brew install mysql
    For MySQL server.

  brew install mysql-connector-c
    For MySQL client libraries only.

  brew install postgresql
    For PostgreSQL server.

We don't install these for you when you install this formula, as
we don't know which datasource you intend to use.
==> Summary
🍺  /usr/local/Cellar/sphinx/2.1.3: 16 files, 18M, built in 17 seconds
{% endcodeblock %}

After that you can install Thinking Sphinx:

{% codeblock Gemfile lang:ruby %}
gem 'mysql2',          '0.3.13'
gem 'thinking-sphinx', '3.0.6'
{% endcodeblock %}

And everything should be ok!

