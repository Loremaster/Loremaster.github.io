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

###capistrano-notifier

If you use heroku then you receive email when deploy finishes. A like this feature and it is very cool that you can have the same with capistrano. So, meet the second hero: [capistrano-notifier](https://github.com/cramerdev/capistrano-notifier).

The superpower of this hero is sending email with commits which has been deployed, for example:

```
Obi-Wan Kenobi deployed
Power branch
stage to
staging on
02/24/2014 at
02:37 PM MSK

https://github.com/jedis/power/compare/8fc335c...e3a999c
8e8b39c The force is strong with this one (Serj L)
b17e255 New lightsaber
```

Looks pretty cool, huh? To install gem add it:

{% codeblock Gemfile lang:ruby %}
group :development do
  gem 'capistrano-notifier'
end
{% endcodeblock %}

After that you should add some settings to be able to send mail. Here is example for gmail:

{% codeblock config/deploy.rb lang:ruby %}
set :notifier_mail_options, {
  :method => :smtp,
  :from   => 'capistrano@domain.com',
  :to     => ['john@doe.com', 'jane@doe.com'],
  :github => 'MyCompany/project-name',
  :smtp_settings => {
    address: "smtp.gmail.com",
    port: 587,
    domain: "gmail.com",
    authentication: "plain",
    enable_starttls_auto: true,
    user_name: MY_USERNAME,
    password: MY_PASSWORD
  }
}
{% endcodeblock %}

After that each person, whose email has been provided in `:to` section will receive the email after deploy. Of course, you man need some time to set thins up properly, but afterwards this feature may be very handy for a lot of people.
