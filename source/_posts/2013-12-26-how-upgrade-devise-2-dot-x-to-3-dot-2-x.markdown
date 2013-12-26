---
layout: post
title: "How upgrade Devise 2 to 3.2"
date: 2013-12-26 15:37:53 +0400
comments: true
categories:
---

[Devise](https://github.com/plataformatec/devise) is a great gem! It really helps you, taking all authentication jobs away from you. However, upgrading Devise up to version 3.x can be a little bit tricky. In this post I'll show you how to do that.

First of all, update your gem via `bundle update` OR specify version in your Gemfile:

{% codeblock Gemfile lang:ruby %}
gem "devise", "~> 3.2.2"
{% endcodeblock %}

And run `bundle install`.

Since devise 3.1 platformatec [announced](http://blog.plataformatec.com.br/2013/08/devise-3-1-now-with-more-secure-defaults/) few security improvements. One of them is `secret_key`. To add it open devise config and add:

{% codeblock config/initializers/devise.rb lang:ruby %}
# The secret key used by Devise. Devise uses this key to generate
# random tokens. Changing this key will render invalid all existing
# confirmation, reset password and unlock tokens in the database.
config.secret_key = '2710f15f11771d6692a3015d7e3dba2cb05539c1f72i6u345df5433hg535kj5x56v6er56if2566c63c2ad670d6859e536b40d87e6543b115609f0464bdd99502abbe241c4'
{% endcodeblock %}

Of course, you should use your own secret key, so change my example to be more secure.

If you have ever generated devise's views, then you should change it's mailers to use `@token` instead of `@resource.*_token`:

{% codeblock views/devise/mailer/confirmation_instructions.html.haml lang:haml %}
# ...
= link_to t('devise.mailer.confirmation_instructions.submit'), confirmation_url(@resource, confirmation_token: @token)
{% endcodeblock %}

{% codeblock views/devise/mailer/reset_password_instructions.html.haml lang:haml %}
# ...
= link_to t('devise.mailer.reset_password_instructions.reset_link'), edit_password_url(@resource, reset_password_token: @token)
{% endcodeblock %}

{% codeblock views/devise/mailer/unlock_instructions.html.haml lang:haml %}
# ...
= link_to t('devise.mailer.unlock_instructions.unlock_link'), unlock_url(@resource, unlock_token: @token)
{% endcodeblock %}

After that everything should works. But take a look into your terminal: you may have deprecation errors, which you should fix!


