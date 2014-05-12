---
layout: post
title: "How thor can help you with your tests (or integration of parallel_tests and cucumber)"
date: 2014-05-12 18:05:53 +0400
comments: true
categories:
---

##Specs it!

Today I'd like to talk about one important (kind of) area of each RoR developer. This is testing. You may wonder, "Why the heck do I even need to write these stupid tests?”. Well, first of all, you may do it because your boss wants it. Or because all other rubyists write them. What do you mean, "I'm not satisfied?”. All right, how about making sure that your app don't crush each time you commit new feature or fix a bug? Tests also help you to upgrade libs, install new Rails and other funny stuff, FYI. I know the company, who can't upgrade to Rails 4, because it's app don't have tests.

Let me suppose, that you are satisfied by that and you started to write tests... Firstly with Rspec, because test-unit is not popular solution. And then you open Cucumber for yourself. And write more and more tests. And in one day you look in your terminal and you start to think that waiting 10 minutes until your features'd finish isn't a good idea, because you can already drink few cups of tea or coffee and with such progress soon you'll be able to watch a "Big bang theory”.

{% img center /images/thor-and-parallel-tests/cucumber-running-xkcd.png %}

<!-- more -->

And you decide to make your tests faster. So, you google for a while and find [parallel_tests](https://github.com/grosser/parallel_tests). You try it and see that your tests run *n* times faster! Wow!

You start use it, and use, it and use it. But after a while some of your tests *suddenly* start to fail from time to time. You fight with it, even add hacks: `sleep 100500`. And even with that some tests may fail randomly from time to time. Oh...

{% img center /images/thor-and-parallel-tests/doge-wow-cucumber.jpeg %}

Of course, you still should try to fix it. But fixing randomly failing piece of code is one of the hardest things that you can ever meet in your live as a programmer, in my humble opinion.

##Thor with parallel

So, I decided to create a thor task for people, who use [parallel_tests](https://github.com/grosser/parallel_tests). This tool do this:

1. Prepare your databases in parallel.
2. Run your features in parallel.
3. Rerun failed tests with vanilla cucumber.

Here is the code:

{% codeblock lib/tasks/parallel.thor lang:ruby %}
require File.expand_path('config/environment.rb')
require 'cucumber/cli/main'
require 'parallel_tests'

class Parallel < Gearup::TasksBase
  desc "and_rerun", 'Prepare and run tests in parallel, and restart failed tests via vanilla cucumber. Run it like that: $ RAILS_ENV=test bundle exec thor parallel:and_rerun'
  def and_rerun
    if Rails.env.test?
      begin
        puts "\nRun: rake parallel:prepare"
        puts `rake parallel:prepare`

        puts "\nRun: rake parallel:features"
        ParallelTests::CLI.new.run(["--type", "cucumber"] + ["features"])
      ensure                                                                                        # Run fail tests if ParallelTests raised error (which it does when some tests fail).
        puts "\nChecking for failed tests..."
        rerun_with_cucumber
      end
    else
      raise "You didn't run task in test environment! Run it like that: $ RAILS_ENV=test bundle exec thor parallel:and_rerun"
    end
  end

  desc "rerun_with_cucumber", 'Run failed tests with vanilla cucumber (non-parallel)'
  def rerun_with_cucumber
    if Rails.env.test?
      parallel_file_with_failed_features = File.join(Rails.root, "tmp", "parallel_cucumber_failures.log")
      failed_features_str = File.open(parallel_file_with_failed_features, "r").read
      tmp_logs = Tempfile.new('parallel_cucumber_failures_tmp', File.join(Rails.root, "tmp"))

      begin
        tmp_logs.write(failed_features_str)
        tmp_logs.rewind

        # Running failed tests if parallel file with failed features contains something.
        unless failed_features_str.empty?
          cucumber_cmd = "@#{tmp_logs.path}"
          puts "\nExecuting failed tests: cucumber #{cucumber_cmd}"
          Cucumber::Cli::Main.new([cucumber_cmd]).execute!
        else
          puts "\nThere are no failed tests."
        end
      ensure
        tmp_logs.close
        tmp_logs.unlink
      end
    else
      raise "You didn't run task in test environment! Run it like that: $ RAILS_ENV=test bundle exec thor parallel:rerun_with_cucumber"
    end
  end
end
{% endcodeblock %}

Well, code contains 2 methods: one to actually rerun failed tests and one to prepare db, run tests in parallel and then call task to run failed features. But this is not all.

{% img center /images/thor-and-parallel-tests/evil-happy-kid.png %}

To enable features to be actually reruned when they fail, you need to enable logging failed features by parallel_tests. This functional has been merged a while ago (wow), but you need to add a specific code to enable it.

Open config/cucumber.yml and add this `--format ParallelTests::Cucumber::FailuresLogger --out tmp/parallel_cucumber_failures.log` to std_options:

{% codeblock config/cucumber.yml lang:ruby %}
#...
std_opts = "--format #{ENV['CUCUMBER_FORMAT'] || 'pretty'} --strict --tags ~@wip --require features --format ParallelTests::Cucumber::FailuresLogger --out tmp/parallel_cucumber_failures.log”
#...
{% endcodeblock %}

And after that you may run the thor task. Failed tests will be stored in `tmp/parallel_cucumber_failures.log` (where you can view them, if you want). To run the thor task print this:

{% codeblock Terminal lang:bash %}
$ RAILS_ENV=test bundle exec thor parallel:and_rerun
{% endcodeblock %}

This thor task has tested under:

* Rails 4
* thor 0.18.1
* parallel_tests 0.16.8
* cucumber-rails 1.4.0
* capybara 2.2.1.

If your cucumber or parallel_tests has lower version than the listed ones, then there is a chance that my code wouldn't work (make sure that you have cucumber at least **1.4.0**).

##Task's internals

Now, let me talk a little about the code. You can skip this section if you are just planning to use the task and nothing more, but I encourage you to read more!

First of all, let me look in the source code of `rerun_with_cucumber` task.

{% codeblock lib/tasks/parallel.thor lang:ruby %}
  # ...
  desc "rerun_with_cucumber", 'Run failed tests with vanilla cucumber (non-parallel)'
  def rerun_with_cucumber
    if Rails.env.test?
      parallel_file_with_failed_features = File.join(Rails.root, "tmp", "parallel_cucumber_failures.log")
      failed_features_str = File.open(parallel_file_with_failed_features, "r").read
      tmp_logs = Tempfile.new('parallel_cucumber_failures_tmp', File.join(Rails.root, "tmp"))

      begin
        tmp_logs.write(failed_features_str)
        tmp_logs.rewind

        # Running failed tests if parallel file with failed features contains something.
        unless failed_features_str.empty?
          cucumber_cmd = "@#{tmp_logs.path}"
          puts "\nExecuting failed tests: cucumber #{cucumber_cmd}"
          Cucumber::Cli::Main.new([cucumber_cmd]).execute!
        else
          puts "\nThere are no failed tests."
        end
      ensure
        tmp_logs.close
        tmp_logs.unlink
      end
    else
      raise "You didn't run task in test environment! Run it like that: $ RAILS_ENV=test bundle exec thor parallel:rerun_with_cucumber"
    end
  end
end
{% endcodeblock %}

A lot of code for such task, isn't it? First of all, you can see that it check this:

{% codeblock lib/tasks/parallel.thor lang:ruby %}
if Rails.env.test?
{% endcodeblock %}

Why? Because otherwise you may lose all your development data.

{% img center /images/thor-and-parallel-tests/lose-local-meme.png %}

Your task'll run with `RAILS_ENV=development`, because this is your current environment. This is the first thing. The second one is that you probably have [database_cleaner](https://github.com/bmabey/database_cleaner) or you clean your test data by your own script in your tests. So, when you'll run your test with your current ENV (as development), you'll lose all data during your tests execution. That is why it's so important to specify `RAILS_ENV=test` when you execute the task and that is why the code is checking this case. It just protects you.

Second note. You may notice that code create temporary file and that code using this temp file to run failed tests. Why? As you may remember, I placed logger in the standard options in cucumber.yml. So, if you run your cucumber to execute failed tests, you'll receive nothing, and list of failed tests'll be immediately erased. To avoid that problem temporary file is using.

And the third note. You may wonder, why I call cucumber and parallel_tests directly and why I don't use ruby's `` `cmd` `` for that? Well, answer is simple. Because `` `cmd` `` in current case just sucks. First of all, you'll lose all colours in the terminal. Second thing is that `` `cmd` `` will return results only after command's finish. That means, if you have few slow tests then you'll just look at blank output for few minutes. And your colleagues'd think that you are crazy. And you don't want that, even if you are crazy. Because, you know, you'd listen stupid jokes about it and etc.

That is why the code uses direct call to the API. It gives you all advantages: colours, live execution, less stupid jokes. It behaves like it should.

That is all for today. Peace.



