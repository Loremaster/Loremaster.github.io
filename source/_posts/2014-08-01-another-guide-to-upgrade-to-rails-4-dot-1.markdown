---
layout: post
title: "Another guide to upgrade to Rails 4.1"
date: 2014-08-01 16:03:05 +0400
comments: true
categories:
---

Rails 4.1 is pretty nice release, but upgrade to this version may be painful. In this post I'll try to help you.

First of all, you need to install the latest version of Rails.

{% codeblock Gemfile lang:ruby %}
gem 'rails', '4.1.2'
gem 'sass-rails', '~> 4.0.3'
{% endcodeblock %}

After that read [official rails guide to upgrade to rails 4.1](http://guides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-from-rails-4-0-to-rails-4-1), because I don't want to talk about these details. Just don't touch section about Spring, I'll tell what to do with that.

<!-- more -->

###Shoulda-matchers

If you use `shoulda-matchers`, then you may see errors in your specs. To make shoulda-matchers work properly you need to install at least this version:

{% codeblock Gemfile lang:ruby %}
gem 'shoulda-matchers', '~> 2.6.1', require: false
{% endcodeblock %}

Only this version (and higher) is compatible with Rails 4.1. After running:

{% codeblock Terminal lang:bash %}
$ bundle
{% endcodeblock %}

You also need to replace in spec_helper.rb this line:

{% codeblock spec_helper.rb lang:ruby %}
require 'shoulda/matchers/integrations/rspec'
{% endcodeblock %}

By these two:

{% codeblock spec_helper.rb lang:ruby %}
require "shoulda/matchers"
require "shoulda/matchers/integrations/rspec"
{% endcodeblock %}

This fix should be enough.

###Bullet

Bullet also generates some errors. Solution is very easy:

{% codeblock Gemfile lang:ruby %}
gem 'bullet', '~> 4.9.0'
{% endcodeblock %}

###Squel

Well, by the time when I was doing the upgrade everything was [baaaaaaad](https://github.com/activerecord-hackery/squeel/issues/307). So I decided to remove the Squeel to exclude this dependency and to finally make the upgrade. Nowadays Squeel [supports](https://github.com/activerecord-hackery/squeel/pull/317) Rails 4.1 (and as I know Rails 4.2 alpha). Yep, I am so lucky that when I has removed Squeel from the project, then it received Rails 4.1 support. Such fail. So 'lucky'. Wow.

###Acts As Taggable

You may have such exceptions:

```
wrong number of arguments (1 for 2)
```

For the method `tagged_with`. To solve this problem you need to update the gem version and run new migration:

{% codeblock Gemfile lang:ruby %}
gem 'acts-as-taggable-on', '~> 3.2.6'
{% endcodeblock %}

{% codeblock Terminal lang:bash %}
$ bundle
$ rake acts_as_taggable_on_engine:install:migrations
$ rake db:migrate
{% endcodeblock %}

###Thinking Sphinx

Comment the thinking sphinx gem after the upgrade and bundle:

{% codeblock Gemfile lang:ruby %}
#gem 'thinking-sphinx'
{% endcodeblock %}

{% codeblock Terminal lang:bash %}
$ bundle
{% endcodeblock %}

Then install this specific version from the brunch on github:

{% codeblock Gemfile lang:ruby %}
gem 'thinking-sphinx', '~> 3.1.1', git: 'git://github.com/pat/thinking-sphinx.git', branch: 'develop', ref: 'c8f549f689'
{% endcodeblock %}

This version includes many fixes for Rails 4.1, including fixes for polymorphic associations (yep, I found few bugs and posted them to Pat, he fixed everything. And this is awesome!).

###Rails-admin

If you use `0.6.2` version and Rails 4.1.2 then you'll meet this exception if you'll try to create record in admin's panel:

```
ActiveModel::ForbiddenAttributesError
```

Because new version hasn't been released yet, you need to install specific version from master:

{% codeblock Gemfile lang:ruby %}
gem 'rails_admin', '0.6.2', git: 'git://github.com/sferik/rails_admin.git', branch: 'master', ref: '02ba1dab32d'
{% endcodeblock %}

###_attributes

Well, I think a lot of people have ever used nested attributes. So, if you'll create nested params in form manually then this topic may be interesting for you. The thing is that you can't pass data like in that way:

```ruby
{some_data: {...}}
```

You need to pass it by this way:

```ruby
{some_data_attributes: {...}}
```

Yes, it's Rails-way and bla-bla-bla. But the thing is that key `some_data` WORKED somehow before, which is weird. But the most weird thing is that if you'll continue to use it then you'll receive this super weird exception:

```
Expected SomeData but received Array.
```

So, now you know what that mean. I wasted A LOT of hours trying to debug that.

###Spring

Spring is app preloader, which is built in Rails core now. It allows you to speed up running task (such as Rails console), which is very nice. Installation is simple:

{% codeblock Gemfile lang:ruby %}
group :development do
  gem 'spring'
end
{% endcodeblock %}

{% codeblock Terminal lang:bash %}
$ bundle
$ bundle exec spring binstub --all
{% endcodeblock %}

After that you can install binstubs for cucumber and rspec. You know, just for fun (even if you don't have them, ha):

{% codeblock Gemfile lang:ruby %}
group :development do
  gem 'spring-commands-rspec'
  gem 'spring-commands-cucumber'
end
{% endcodeblock %}

{% codeblock Terminal lang:bash %}
$ bundle exec spring binstub cucumber
$ bundle exec spring binstub rspec
$ spring stop
{% endcodeblock %}

Now you can run your tests. Don't forger to run them via `bin/rspec` and `bin/cucumber` to use Spring. There is one gotcha with Spring, which you can read about in the next section.

###Parallel_tests

Here is the gotcha. It doesn't work with Spring. It's very, very sad. Why? Because parallel_test is big help for you tests. Read [here more why it awesome](https://github.com/grosser/parallel_tests).

So, you'll get such exceptions:

```
Failure/Error: Unable to find matching line from backtrace
     ActiveRecord::StatementInvalid:
       PG::TRDeadlockDetected: ERROR:  deadlock detected
       LINE 1: SELECT  "contact_types".* FROM "contact_types"  WHERE "conta...
                                              ^
       DETAIL:  Process 70359 waits for AccessShareLock on relation 24882218 of database 7594218; blocked by process 70358.
       Process 70358 waits for AccessExclusiveLock on relation 24888649 of database 7594218; blocked by process 70359.
       HINT:  See server log for query details.
```

But there is workaround! Open `config/spring.rb` and paste:

{% codeblock config/spring.rb lang:ruby %}
require "active_support/core_ext/module/aliasing"
require "spring/application"

class Spring::Application
  def connect_database_with_reconfigure_database
    disconnect_database
    reconfigure_database
    connect_database_without_reconfigure_database
  end

  alias_method_chain :connect_database, :reconfigure_database

  def reconfigure_database
    if active_record_configured?
      ActiveRecord::Base.configurations = Rails.application.config.database_configuration
    end
  end
end
{% endcodeblock %}

Then stop the spring, so he could see these code above:

{% codeblock Terminal lang:bash %}
$ spring stop
{% endcodeblock %}

And now you run tests in parallel.

###Strong Params

I don't know how many people may meet my situation, but anyway. If you build your nested attributes hash in form manually, then (and only then) there is possible situation when keys in your hash may contain hashes and/or integer. For example:

```ruby
{:service =>
  {:shipping_locations_attributes =>
    {“w234pxc453”=>
      {:name => "LA, USA",
       :longitude => "37.6173",
       :latitude => "32.755826"},
    "0"=>
      {:name => "LC, USA",
       :longitude => "12.6173",
       :latitude => "58.755826"}
       }
  }
}
```

In my case there is 3 possible situations:

1. All keys are integers.
2. All keys are hashes.
3. Keys are hashes AND integers.

You may think that this would work:

```ruby
params.require(:service).permit(:type_professional).tap do |whitelist|
  whitelist[:shipping_locations_attributes] = params[:service][:shipping_locations_attributes]
end
```

Unfortunately, it didn't help me (but it may help you). That is why I have created special method, which knows about these 3 possible situations and it generates array with this data dynamically, so you just insert this data in `permit`.

```ruby
  # Pair keys (which StronParams doesn't allow to use for nested attributes) with data to permit.
  # Input:
  #   keys - Array - keys, which need to be appear in result.
  #   data_to_permit - Array/Integer/String/etc - data, which need to be in result as value.
  #
  # Return:
  #   1) Array with hash, where keys is input keys and values are duplicated data_to_permit IF all keys are not integers.
  #   2) data_to_permit - if all keys are integer (otherwise data will not be permitted).
  #
  # Example:
  #   pair_random_keys_with_data_to_permit(["1a0f08fcbc", "345rec32c1"], [:id, :desc, :number])
  #   # => [{:"1a0f08fcbc"=>[:id, :desc, :number], :"345rec32c1"=>[:id, :desc, :number]}]
  #
  #   pair_random_keys_with_data_to_permit(["0", "1"], [:id, :desc, :number])
  #   # => [:id, :desc, :number]
  def pair_random_keys_with_data_to_permit(keys, data_to_permit)
    return [] if keys.nil? || data_to_permit.nil?

    keys_integers = keys.all?{ |key| key.to_i.to_s == key }

    # When keys are all integers, then we should return data_to_permit, Rails's StrongParams
    # will take care of it.
    if keys_integers
      data_to_permit
    else
      array_for_hash_to_permit = keys.map{ |key| [key, data_to_permit] }                          # Pair each key with data to permit.
      [Hash[array_for_hash_to_permit].symbolize_keys]                                             # Array with hash where keys are symbols (if they may be symbols).
    end
  end
```

Here is the example how to use this method. You just pass a data to the method and then you should use it's results to permit it.

{% codeblock app/controllers/services_controller.rb lang:ruby %}
  def service_params
    shipping_locations_keys = params[:service][:shipping_locations_attributes].try(:keys)
    shipping_locations_allowed_attributes = [:id, :name, :longitude, :latitude]
    shipping_locations_attrs = pair_random_keys_with_data_to_permit(shipping_locations_keys, shipping_locations_allowed_attributes)

    params.require(:service).permit(:type_professional, shipping_locations_attributes: shipping_locations_attrs)
  end
{% endcodeblock %}

I hope it'll help somebody.

###Rails 4.1.5 notes

When you upgrade your app to Rails 4.1.5 and you use Devise for authentication, then you can met this problem, when user tries to resend password:

{% codeblock Terminal lang:bash %}
ActiveModel::ForbiddenAttributesError - ActiveModel::ForbiddenAttributesError:
  activemodel (4.1.5) lib/active_model/forbidden_attributes_protection.rb:21:in `sanitize_for_mass_assignment'
  activerecord (4.1.5) lib/active_record/relation/query_methods.rb:568:in `where!'
  activerecord (4.1.5) lib/active_record/relation/query_methods.rb:559:in `where'
  activerecord (4.1.5) lib/active_record/querying.rb:10:in `where'
  app/models/user.rb:83:in `find_first_by_auth_conditions'
  devise (3.2.4) lib/devise/models/authenticatable.rb:260:in `find_or_initialize_with_errors'
  devise (3.2.4) lib/devise/models/recoverable.rb:99:in `send_reset_password_instructions'
  app/controllers/passwords_controller.rb:4:in `create'
  actionpack (4.1.5) lib/action_controller/metal/implicit_render.rb:4:in `send_action'
  actionpack (4.1.5) lib/abstract_controller/base.rb:189:in `process_action'
  actionpack (4.1.5) lib/action_controller/metal/rendering.rb:10:in `process_action'
  actionpack (4.1.5) lib/abstract_controller/callbacks.rb:20:in `block in process_action'
  activesupport (4.1.5) lib/active_support/callbacks.rb:113:in `call'
  activesupport (4.1.5) lib/active_support/callbacks.rb:149:in `block in halting_and_conditional'
  activesupport (4.1.5) lib/active_support/callbacks.rb:229:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:229:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:166:in `block in halting'
  activesupport (4.1.5) lib/active_support/callbacks.rb:86:in `run_callbacks'
  actionpack (4.1.5) lib/abstract_controller/callbacks.rb:19:in `process_action'
  actionpack (4.1.5) lib/action_controller/metal/rescue.rb:29:in `process_action'
  actionpack (4.1.5) lib/action_controller/metal/instrumentation.rb:31:in `block in process_action'
  activesupport (4.1.5) lib/active_support/notifications.rb:159:in `block in instrument'
  activesupport (4.1.5) lib/active_support/notifications/instrumenter.rb:20:in `instrument'
  activesupport (4.1.5) lib/active_support/notifications.rb:159:in `instrument'
  actionpack (4.1.5) lib/action_controller/metal/instrumentation.rb:30:in `process_action'
  actionpack (4.1.5) lib/action_controller/metal/params_wrapper.rb:250:in `process_action'
  activerecord (4.1.5) lib/active_record/railties/controller_runtime.rb:18:in `process_action'
  actionpack (4.1.5) lib/abstract_controller/base.rb:136:in `process'
  actionview (4.1.5) lib/action_view/rendering.rb:30:in `process'
  actionpack (4.1.5) lib/action_controller/metal.rb:196:in `dispatch'
  actionpack (4.1.5) lib/action_controller/metal/rack_delegation.rb:13:in `dispatch'
  actionpack (4.1.5) lib/action_controller/metal.rb:232:in `block in action'
  actionpack (4.1.5) lib/action_dispatch/routing/route_set.rb:82:in `dispatch'
  actionpack (4.1.5) lib/action_dispatch/routing/route_set.rb:50:in `call'
  actionpack (4.1.5) lib/action_dispatch/routing/mapper.rb:45:in `call'
  actionpack (4.1.5) lib/action_dispatch/journey/router.rb:71:in `block in call'
  actionpack (4.1.5) lib/action_dispatch/journey/router.rb:59:in `call'
  actionpack (4.1.5) lib/action_dispatch/routing/route_set.rb:678:in `call'
  rack-pjax (0.7.0) lib/rack/pjax.rb:12:in `call'
  newrelic_rpm (3.8.1.221) lib/new_relic/rack/error_collector.rb:55:in `call'
  newrelic_rpm (3.8.1.221) lib/new_relic/rack/agent_hooks.rb:32:in `call'
  newrelic_rpm (3.8.1.221) lib/new_relic/rack/browser_monitoring.rb:27:in `call'
  newrelic_rpm (3.8.1.221) lib/new_relic/rack/developer_mode.rb:45:in `call'
  meta_request (0.3.0) lib/meta_request/middlewares/app_request_handler.rb:13:in `call'
  rack-contrib (1.1.0) lib/rack/contrib/response_headers.rb:17:in `call'
  meta_request (0.3.0) lib/meta_request/middlewares/headers.rb:16:in `call'
  meta_request (0.3.0) lib/meta_request/middlewares/meta_request_handler.rb:13:in `call'
  bullet (4.10.0) lib/bullet/rack.rb:12:in `call'
  simple_captcha2 (0.3.2) lib/simple_captcha/middleware.rb:26:in `call'
  warden (1.2.3) lib/warden/manager.rb:35:in `block in call'
  warden (1.2.3) lib/warden/manager.rb:34:in `call'
  rack (1.5.2) lib/rack/etag.rb:23:in `call'
  rack (1.5.2) lib/rack/conditionalget.rb:35:in `call'
  rack (1.5.2) lib/rack/head.rb:11:in `call'
  remotipart (1.2.1) lib/remotipart/middleware.rb:27:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/params_parser.rb:27:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/flash.rb:254:in `call'
  rack (1.5.2) lib/rack/session/abstract/id.rb:225:in `context'
  rack (1.5.2) lib/rack/session/abstract/id.rb:220:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/cookies.rb:560:in `call'
  activerecord (4.1.5) lib/active_record/query_cache.rb:36:in `call'
  activerecord (4.1.5) lib/active_record/connection_adapters/abstract/connection_pool.rb:621:in `call'
  activerecord (4.1.5) lib/active_record/migration.rb:380:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/callbacks.rb:29:in `block in call'
  activesupport (4.1.5) lib/active_support/callbacks.rb:82:in `run_callbacks'
  actionpack (4.1.5) lib/action_dispatch/middleware/callbacks.rb:27:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/reloader.rb:73:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/remote_ip.rb:76:in `call'
  better_errors (1.1.0) lib/better_errors/middleware.rb:84:in `protected_app_call'
  better_errors (1.1.0) lib/better_errors/middleware.rb:79:in `better_errors_call'
  better_errors (1.1.0) lib/better_errors/middleware.rb:56:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/debug_exceptions.rb:17:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/show_exceptions.rb:30:in `call'
  railties (4.1.5) lib/rails/rack/logger.rb:38:in `call_app'
  railties (4.1.5) lib/rails/rack/logger.rb:20:in `block in call'
  activesupport (4.1.5) lib/active_support/tagged_logging.rb:68:in `block in tagged'
  activesupport (4.1.5) lib/active_support/tagged_logging.rb:26:in `tagged'
  activesupport (4.1.5) lib/active_support/tagged_logging.rb:68:in `tagged'
  railties (4.1.5) lib/rails/rack/logger.rb:20:in `call'
  quiet_assets (1.0.3) lib/quiet_assets.rb:23:in `call_with_quiet_assets'
  actionpack (4.1.5) lib/action_dispatch/middleware/request_id.rb:21:in `call'
  request_store (1.0.6) lib/request_store/middleware.rb:8:in `call'
  rack (1.5.2) lib/rack/methodoverride.rb:21:in `call'
  rack (1.5.2) lib/rack/runtime.rb:17:in `call'
  activesupport (4.1.5) lib/active_support/cache/strategy/local_cache_middleware.rb:26:in `call'
  rack (1.5.2) lib/rack/lock.rb:17:in `call'
  actionpack (4.1.5) lib/action_dispatch/middleware/static.rb:64:in `call'
  rack (1.5.2) lib/rack/sendfile.rb:112:in `call'
  railties (4.1.5) lib/rails/engine.rb:514:in `call'
  railties (4.1.5) lib/rails/application.rb:144:in `call'
  rack (1.5.2) lib/rack/content_length.rb:14:in `call'
  thin (1.6.2) lib/thin/connection.rb:86:in `block in pre_process'
  thin (1.6.2) lib/thin/connection.rb:84:in `pre_process'
  thin (1.6.2) lib/thin/connection.rb:53:in `process'
  thin (1.6.2) lib/thin/connection.rb:39:in `receive_data'
  eventmachine (1.0.3) lib/eventmachine.rb:187:in `run'
  thin (1.6.2) lib/thin/backends/base.rb:73:in `start'
  thin (1.6.2) lib/thin/server.rb:162:in `start'
  rack (1.5.2) lib/rack/handler/thin.rb:16:in `run'
  rack (1.5.2) lib/rack/server.rb:264:in `start'
  railties (4.1.5) lib/rails/commands/server.rb:69:in `start'
  railties (4.1.5) lib/rails/commands/commands_tasks.rb:81:in `block in server'
  railties (4.1.5) lib/rails/commands/commands_tasks.rb:76:in `server'
  railties (4.1.5) lib/rails/commands/commands_tasks.rb:40:in `run_command!'
  railties (4.1.5) lib/rails/commands.rb:17:in `<top (required)>'
  bin/rails:8:in `<top (required)>'
  spring (1.1.3) lib/spring/client/rails.rb:27:in `call'
  spring (1.1.3) lib/spring/client/command.rb:7:in `call'
  spring (1.1.3) lib/spring/client.rb:26:in `run'
  spring (1.1.3) bin/spring:48:in `<top (required)>'
  spring (1.1.3) lib/spring/binstub.rb:11:in `<top (required)>'
  bin/spring:16:in `<top (required)>'
  bin/rails:3:in `<main>'
{% endcodeblock %}

In my case I had overridden Devise's controller:

{% codeblock app/controllers/passwords_controller.rb lang:ruby %}
class PasswordsController < Devise::PasswordsController
  def create
    @user = resource_class.send_reset_password_instructions(resource_params)

    respond_to do |format|
      format.js
    end
  end
end
{% endcodeblock %}

Fix is pretty easy. We need to permit email:

{% codeblock app/controllers/passwords_controller.rb lang:ruby %}
class PasswordsController < Devise::PasswordsController
  def create
    @user = resource_class.send_reset_password_instructions(resource_params.permit(:email))

    # ...
  end
end
{% endcodeblock %}

###Last words

Check out [railsdiff](http://railsdiff.org/diff/v4.0.5/v4.1.5/) to view changes in the code. Don't forget to bump you gems. Make sure that all tests are green. Have a good test coverage and don't bump your app without it.
