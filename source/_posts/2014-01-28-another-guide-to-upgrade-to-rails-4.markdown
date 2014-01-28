---
layout: post
title: "Another guide to upgrade to Rails 4"
date: 2014-01-28 15:53:10 +0400
comments: true
categories:
---

Yep, this is another guide, which will help you, i hope, to upgrade your app to Rails 4. Before start, please make sure that you are running latest Rails 3 release, i think it is 3.2.16 nowadays. Yes, i'm sorry, Rails 2 guys, but this guide'll not help a lot, you have a lot of work to do, sorry! :)

<!-- more -->

What should you do next? You should upgrade **all** of your gems to their latest versions for rails 3, which can be a little bit hard, but good test coverage will help you a lot! If you don't have test coverage, then take a deep breath and… And create your test suit (i prefer Rspec for unit tests and Capybara, Capybara-webkit and Cucumber - for integration tests). You don't have to get 100% test coverage for your app, covering a main logic should be enough. Believe me, it's very important - at least you'll be sure that libs upgrade and fresh code base'll not break your app.

Make sure that you read `changelog` (this file also may be known as `history`) of each gem to get to know if gem'll break anything. Good news is that upgrading of gems is not Rails 4 specific, so you can deploy your app when your gems'll be up to date.

When your upgrade of gems is complete and all tests pass then you may push updated app to your test server and after testing to the production (it depends, of course, how your system is set up). Then you should wait for some time (day or more) to make sure that everything works great and upgrade of libs hasn't brake anything. Yes, tests may miss something, its okey, just write new tests and fix the problem to avoid regressions.

When you upgrade gems then you should make a note for yourself where you should specify which of the gems have new version for Rails 4 and which gems are not working in Rails 4 (so you may remove them after upgrade). Here is my example:

```
Rails 4 gems versions:
rails_admin 0.6.0
rails-i18n 4.0.1
simple form 3.0
paper_trail 3

Remove in rails 4:
activerecord-postgres-hstore
sextant
turbo-sprockets-rails3
```

##Are gems ready for rails 4?

Next step is to check out [ready4rails4](http://ready4rails4.net/), this site allow you to upload your Gemfile and check if your app is fully ready for Rails 4. Of course, you may not find your gems statusesor even gems. It's okay, you should check out repos of these gems (the easiest way is to find in on [rubygems](http://rubygems.org/)) and then search in Readme, History/Changes or issues for any mentions of Rails 4. If you don't find anything, then you should just create an issue and ask about Rails 4 support. Actually, if gem is supported by an author, then it may have support for Rails 4. If repo is abandoned for a long time then you should probably search for forks or alternatives via [ruby-toolbox](https://www.ruby-toolbox.com/) or create your own fork.

If you found any data about Rails 4's support for your gem/gems then you should definitely post your discover on [ready4rails4](http://ready4rails4.net/), it'll help other people a lot!

All these steps can take a long time, because you may have a lot gems with unknown or even “not ready” statuses so you'll do a long research. But i'm absolutely sure that you must do these preparation steps, because it'll really help you a lot during the actual rails 4 upgrade.

##Actual bump of rails

So, you did all these steps and now you are ready to bump. Open the `Gemfile`:

{% codeblock Gemfile lang:ruby %}
gem 'rails', '4.0.2'

gem 'sass-rails',   '~> 4.0.0'
gem 'coffee-rails', '~> 4.0.0'
gem 'uglifier', '>= 1.3.0'

# add these gems to help with the transition:
gem 'protected_attributes'
gem 'rails-observers'
gem 'actionpack-page_caching'
gem 'actionpack-action_caching'
gem 'activerecord-session_store'
{% endcodeblock %}

Yeap, you should remove assets group, bump rails, sass-rails, coffee-rails and uglifier. And you want to add gems above to have smooth transition. As you remember, i asked you to make a list of gems which have their own version for Rails 4 and which should be removed. It's probably a good time to specify their version and remove unused gems according to your list. After that run `bundle install` and make sure that bundle'll evaluate and not throw errors.

Here i want to add my two cents about [cancan](https://github.com/ryanb/cancan). This gem is a great tool and i'm sure that you may use it in your app. Well, here are good news: it's latest version (1.6.10) works if you keep attr_accessible (via “protected_attributes” gem). This also means that you shouldn't remove some options in your configs and you shouldn't start to use StrongParams (which is used by a lot of gems with Rails 4 support). So, you should keep these options in your configs:

{% codeblock config/application.rb lang:ruby %}
# Enforce whitelist mode for mass assignment.
# This will create an empty whitelist of attributes available for mass-assignment for all models
# in your app. As such, your models will need to explicitly whitelist or blacklist accessible
# parameters by using an attr_accessible or attr_protected declaration.
config.active_record.whitelist_attributes = true
{% endcodeblock %}

{% codeblock config/environments/* lang:ruby %}
# Raise exception on mass assignment protection for Active Record models
config.active_record.mass_assignment_sanitizer = :strict
{% endcodeblock %}

Here `config/environments/*` means all filed in that directory. That should be enough for cancan to work properly with Rails 4 until you'd move to StrongParams completely.

But i want to warn you about few things. First of all, almost all of the gems which support Rails 4 are not working with `attr_accessible`. It is bad for you because you want to have a smooth transition (who doesn't?). So, you need to fork gems which generate mass-assigment errors and monkey-patch them so they'd work with your app without StrongParams. This is not a bad solution, because if you have a big app with a lot of logic or you just use Cancan then integration of StrongParams'll be pain in the ass.

Of course, if you are not depend on Cancan and/or your app is not so big then you might integrate StrongParams during Rails 4 upgrade and don't think a lot about forks and patches. But even then you may need to patch something (welcome to opensourse)!

I patched few gems so it would work with Rails 4, here is the list:

1. https://github.com/Loremaster/rails_admin - it has problems with paper_trail 3 (which is for rails 4 only), so i patched it.
2. https://github.com/Loremaster/simple-captcha - i added a patch for support `attr_accessible` only in Rails 4, use gem ‘simple_captcha2' if you are on StrongParams.
3. https://github.com/Loremaster/impressionist - i added a patch for support `attr_accessible` only in Rails 4, use original gem ‘impressionist' if you are on StrongParams.

Wow, so much work is done! And I didn't even ask to launch your specs! I guess now its a good time to run it! I suppose that you'll have a ton of deprecation messages and red tests. Check their errors, they may happen because of mostly old gems, or their incapability to work with Rails 4 and attr_accessible. You also may want to check section below about deprecation messages.

Ok, now lets fix configs. Firstly you should edit `config/application.rb` and `config/environmens/*`, check out [railsdiff](http://railsdiff.org/html/v3.2.16-v4.0.2.html) for that purpose. Keep in mind my notes about cancan!

When you finish with `config/` directory then you might want to apply edits to other files. Again, use [railsdiff](http://railsdiff.org/html/v3.2.16-v4.0.2.html) for that. And don't forget to add `secret_key_base` in your secret_token.rb!

After all edits in configs you need to generate `bin` directory:

{% codeblock Terminal lang:bash %}
$ rake rails:update:bin
{% endcodeblock %}

This will generate bin/ folder with bundle, rails and rake. You also might want to generate binstubs for spec and cucumber:

{% codeblock Terminal lang:bash %}
$ bundle binstubs rspec-core
$ bundle binstubs cucumber
{% endcodeblock %}

##Deprecations

I will not mention all possible deprecation messages, but i want to tell you about an important ones.

1) Dynamic methods. Here is short list (old method -> new method):

```ruby
find_all_by_... -> where(...)
find_last_by_... -> where(…).last
scoped_by_... -> where(...)
find_or_initialize_by_... -> where(...).first_or_initialize
find_or_create_by_... -> find_or_create_by(...) or where(...).first_or_create
find_or_create_by_...! -> find_or_create_by!(...) or where(…).first_or_create!
```

2) Match can't be used in routes anymore. So, instead of:

```ruby
match 'home/index', to:'home#index'
```

You should use this one (if want to show page, of course):

```ruby
get 'home/index', to:'home#index'
```

You can use `get`, `post`, `put`, `delete` or even `patch` depending on what you want to achieve.

3) Eager loading table(s) that are referenced in a string SQL snippet. That means that this will work great:

```ruby
Post.includes(:comments).first
```

But that:

```ruby
Post.includes(:comments).where("comments.title = ‘foo'")
```

Should be rewritten to use reference:

```ruby
Post.includes(:comments).where("comments.title = ‘foo'”).references(:comments)
```

Here is note why:

`Currently, Active Record recognizes the table in the string, and knows to JOIN the comments table to the query, rather than loading comments in a separate query. However, doing this without writing a full-blown SQL parser is inherently flawed. Since we don't want to write an SQL parser, we are removing this functionality. From now on, you must explicitly tell Active Record when you are referencing a table from a string`

4) New scopes. You should pass callable object to the scope (lambda, for example). So you need change that:

```ruby
scope :order_by_name, order('brands.name ASC')
```

To that:

```ruby
scope :order_by_name, -> { order('brands.name ASC') }
```

5) `.all` instead of `.scoped`

In Rails 3 you could use Model.scoped to return Rails's relation of all Model's objects. In Rails 4 `.all` returns Rails Relation by default, so you should replace .scoped by `.all`.

##Deployment

I have deployed my apps via capistrano 2.15.5 and i recommend you to use this version or higher. First of all, make sure that your Gemfile doesn't include gems which works with Rails 3 only, for example turbo-sprockets-rails3. In my case, i forgot to remove this gem so i had really weird error during first deploy with Rails 4.

Also, you should use `deploy:finalize_update` instead of `deploy:update_code`, i had an issue in Capistrano 2 with this thing, and this simple fix helped a lot.

And last note. You can get this error during the deploy:

{% codeblock Terminal lang:bash %}
capistrano-2.15.5/lib/capistrano/recipes/deploy/assets.rb:68:in `block (3 levels) in load':
More than one asset manifest file was found in '/srv/stage/shared/assets'.
If you are upgrading a Rails 3 application to Rails 4, follow these instructions:
http://github.com/capistrano/capistrano/wiki/Upgrading-to-Rails-4#asset-pipeline (RuntimeError)
{% endcodeblock %}

This error appears because Rails 3 has its own asset's manifest and Rails 4 has it's own. I just renamed assets directory and created the new one (empty). After that deployment was successful. Also you might just remove manifests and repeat your deploy but it'll make much harder to rollback so i don't recommend you to do that.
