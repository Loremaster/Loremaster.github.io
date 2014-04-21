---
layout: post
title: "Useful gems for Rails developer. Part 1: development"
date: 2014-04-18 17:31:57 +0400
comments: true
categories:
---

##[better_errors](https://github.com/charliesome/better_errors)

We meet different errors almost every day. Sometimes we get exceptions and we want to debug them to find the root of the problem. And better_errors helps us with that. It shows much nicer errors. It is like superman, just in programming area. And it is not just show stack trace to you, it also allows you to interact with your code and data using the console to get feedback immediately. It is nice, and I think, that you will get this feeling soon.

{% img center /images/better-errors-preview.png %}

<!-- more -->

Installation is very simple:

{% codeblock Gemfile lang:ruby %}
group :development do
  gem "better_errors"
  gem "binding_of_caller"
end
{% endcodeblock %}

Yep, these 2 gems! You need second gem to enable advanced features of the better_errors: local/instance variable inspection, pretty stack frame names. Don’t be lazy, just copy and paste them.

And you should use these gems only in development mode, because, well you don’t want other people to access your data, do you?

##[quiet_assets](https://github.com/evrone/quiet_assets)

During our every day work we often look into Rails logs to understand why we don’t get what we want, or to see why page is loading so long (yeah, because of these 100500 joins). But, oh crap… So much time to scroll to the next controller’s response!

{% codeblock log/development.log lang:ruby %}
Started GET "/assets/application.js?body=1" for 127.0.0.1 at 2012-02-13 13:24:04 +0400
Served asset /application.js - 304 Not Modified (8ms)
…
{% endcodeblock %}

I really hate the fact, that vanilla Rails is so noisy with assets. Such long logs. So boring. Wow.

{% img center /images/doge-meme.jpg %}

Thanks to guy… Oh well, it is company. Thanks to guys from the company *Evrone* this pain is over. Install the gem:

{% codeblock Gemfile lang:ruby %}
group :development do
  gem "quiet_assets"
end
{% endcodeblock %}

And restart server. It is just that simple.

##[meta_request](https://github.com/dejan/rails_panel/tree/master/meta_request) & rails_panel

You pressed the "Ok" button and data has been sent to the server. New page appeared. Now you are wondering - why did you actually pressed this button? Oh, yeah, you need to check this edge case and find what server would respond, because last 10 exceptions on production happened because of you. And your boss is not happy!

Okay, you need to open terminal and scroll to this controller. But, oh, wait, first of all where is this icon? Here it is, and let me scroll for few seconds…
Okay, it is a little bit long. It is kinda boring repeat this movement again and again. So, you wondering is a any way to solve that?

Yes! It is rails_panel with meta_request. Idea is simple - you install the extension for your browser and then you install the gem so it would grab your data (and send it to the NSA, of course).

Install the gem:

{% codeblock Gemfile lang:ruby %}
group :development do
  gem "meta_request"
end
{% endcodeblock %}

And now you need install the actual extension. Well, it is for Google Chrome only. What? You use Firefox? Really? Are you hipster? How dare are you not to share your private data and passwords with Google! Install it immediately! And after that use that link: https://chrome.google.com/webstore/detail/railspanel/gjpfobpafnhjhbajcjgccbbdofdckggg. Yeap, I know, that this link is awesome.

##[bullet](https://github.com/flyerhzm/bullet)

As you may know, database request kinda expensive and may take a lot of time to render the page. Of course, it is not always the case, your app may may load so long because you decided to have 100500 share buttons on each page and such buttons loading synchronously.

But I really hope that you don’t do this.

So, for example you have list of items on your page and each item has owner. So you list such data on the page and if open your logs, you will see a lot of requests like this:

{% codeblock log/development.log lang:ruby %}
SELECT "owners".* FROM "owners" WHERE "owners"."id" = $1 ORDER BY "owners"."id" ASC [["id", 1]]
…
{% endcodeblock %}

Lists of such requests kill performance of your app. Users are angry, they leave your site, it generates less money, boss is angry, you are fired… What a nice picture!

{% img center /images/hawkward.png %}

To avoid such awkwardness you definitely need to install the bullet:

{% codeblock Gemfile lang:ruby %}
group: "development" do
  gem "bullet"
end
{% endcodeblock %}

After that you should create the initialiser:

{% codeblock config/environments/development.rb lang:ruby %}
config.after_initialize do
  Bullet.enable = true
  Bullet.alert = true
  Bullet.bullet_logger = true
  Bullet.console = true
  Bullet.growl = true
  Bullet.xmpp = { :account  => 'bullets_account@jabber.org',
                  :password => 'bullets_password_for_jabber',
                  :receiver => 'your_account@jabber.org',
                  :show_online_status => true }
  Bullet.rails_logger = true
  Bullet.airbrake = true
  Bullet.add_footer = true
  Bullet.stacktrace_includes = [ 'your_gem', 'your_middleware' ]
end
{% endcodeblock %}

After that you will get such message:

{% codeblock log/development.log lang:ruby %}
2009-08-25 20:40:17[INFO] N+1 Query: PATH_INFO: /items;    model: Item => associations: [owners]·
Add to your finder: :include => [:owners]
2009-08-25 20:40:17[INFO] N+1 Query: method call stack:·
…
{% endcodeblock %}

So you should do something like this in your controller:

{% codeblock app/conntrolles/items_controller.rb lang:ruby %}
#...
@items = Item.include(:owners)
#...
{% endcodeblock %}

And your app would be much faster. Everybody is happy.

P.S. Good news, everyone! All gems, which are listed here, work with Rails 4! Peace!



