---
layout: post
title: "Enlarge your capistrano"
date: 2014-02-24 14:58:45 +0400
comments: true
categories:
---

I would like to tell you about few gems, which would make your deployment a little bit easier. Well, it worked for me.

###rvm-capistrano

Our first hero on the scene is [rvm-capistrano](https://github.com/wayneeseguin/rvm-capistrano). This hero is special, he prefers only capistrano №2. But time is changing and capistrano №3 is walking on streets of our computers, so, if you have seen it and even use it then you may select another hero: [rvm1-capistrano3](https://github.com/rvm/rvm1-capistrano3#readme).

But I will talk about oldies here. What does this gem actually do? It brings integration between Rvm and capistrano. There are some discusses in the internet what to use, some people prefer [rbenv](https://github.com/sstephenson/rbenv), other use [Rvm](https://rvm.io/). Actually, rbenv is very small and it has very small code base, but it doesn't have power of Rvm, so your choice should depend on your situation. But let me talk about hero, he is waiting to be presented!

Installation is simple, because you already use capistrano, do you?

{% codeblock Gemfile lang:ruby %}
group :development do
  gem 'capistrano'
  gem 'rvm-capistrano'
end
{% endcodeblock %}

Now you should call the hero, to do that just place this line somewhere in the top of your capistrano's deploy file:

{% codeblock config/deploy.rb lang:ruby %}
require 'rvm/capistrano'
{% endcodeblock %}

This will bring to you few things. First of all, it extends `base` to automatically `set :default_shell`. Also it adds this tasks:

* `rvm:info` - show rvm info
* `rvm:list` - list rvm rubies
* `rvm:info_list` - show info and list rubies
* `rvm:install_rvm` - install RVM of the given choice to the server.
* `rvm:install_ruby` - install RVM ruby to the server, create gemset if needed
* `rvm:create_gemset` - create gemset

If you forget about correct syntax you can always run `cap -T rvm` to list all tasks which gem has added. There are some additional tasks which you may include you can check them out in [readme](https://github.com/wayneeseguin/rvm-capistrano#modules).

With this gem you may, for example, update Rvm on each deploy and make sure that ruby is installed:

{% codeblock config/deploy.rb lang:ruby %}
require "rvm/capistrano"

set :rvm_ruby_string, :local        # use the same ruby as used locally for deployment

before 'deploy', 'rvm:install_rvm'  # install/update RVM
before 'deploy', 'rvm:install_ruby' # install Ruby and create gemset (both if missing)
{% endcodeblock %}

Or even restrict Rvm on some servers!

{% codeblock config/deploy.rb lang:ruby %}
set :rvm_require_role, :rvm
require "rvm/capistrano/selector_mixed"
role :rvm, "web1", "web2"
role :app, "web1", "web2", "web3"
{% endcodeblock %}

Check out [readme](https://github.com/wayneeseguin/rvm-capistrano) for more details about this hero.