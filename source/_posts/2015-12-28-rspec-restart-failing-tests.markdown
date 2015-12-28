---
layout: post
title: "Restart failing tests in rspec 3"
date: 2015-12-28 19:16:29 +0300
comments: true
categories:
---

As good developers we write code. Since ruby is so dynamic we need a good test coverage to make sure that our app works. Part of our test suit for web apps is features or integration tests.

Unfortunately integration tests can fail from time to time. It's quite hard to fix such tests. It doesn't mean that we shouldn't fix them but sometimes we need to deal with them in some other way. I like rspec and I use it for the whole test suite. Here I want to show you how can you **restart failed tests**.

<!-- more -->

Developers of rspec recently added this very nice feature which allows you to restart failed specs. First of all make sure that you have Rspec 3.4 or higher:

{% codeblock Gemfile lang:ruby %}
group :development, :test do
  gem 'rspec-rails', '3.4.0'
end
{% endcodeblock %}

Now open your spec_helper and add this line:

{% codeblock spec/spec_helper.rb lang:ruby %}
RSpec.configure do |config|
  config.example_status_persistence_file_path = "#{Rails.root}/spec/specs_with_statuses.txt"
end
{% endcodeblock %}

You need to specify file which will store tests statuses. I recommend to add this file to gitignore:

{% codeblock .gitignore lang:bash %}
/spec/specs_with_statuses.txt
{% endcodeblock %}

Now after running your specs you can run your failing tests:

{% codeblock Terminal lang:bash %}
$ bundle exec rspec
$ bundle exec rspec --only-failures
{% endcodeblock %}

Running spec suit will fill file from example_status_persistence_file_path with tests and their statuses.

You can create simple shell script to run your test suit and rerun failed tests. Here it is:

{% codeblock run_tests.sh lang:bash %}
echo "$ bundle exec rake db:test:prepare"
bundle exec rake db:test:prepare

echo "$ bundle exec rspec"
bundle exec rspec

echo "$ bundle exec rspec --only-failures"
bundle exec rspec --only-failures
{% endcodeblock %}

It can be useful if you use CI server where you can just point to run this shell script. It can be also quite useful to use locally to re-rerun failures automatically instead of typing command to every time.

I hope this article was useful for you!
