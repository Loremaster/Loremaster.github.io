---
layout: post
title: "How integrate ActsAsTaggableOn with Jquery Token Input (with Rails 3)"
date: 2013-12-31 16:53:11 +0400
comments: true
categories:
---

Have you ever met problems with integrating the [Jquery Token Input](http://loopj.com/jquery-tokeninput/) and the [acts_as_taggable_on](https://github.com/mbleigh/acts-as-taggable-on)? Or are you gonna to implement these things together? Hold on! This article'll help you: I'll show you step by step how to do that with Rails 3.2.

<!-- more -->

##Acts as taggable on

First of all, you should, of course, install acts_as_taggable_on. Add this in your gemfile:

{% codeblock Gemfile lang:ruby %}
gem 'acts-as-taggable-on'
{% endcodeblock %}

And run these commands to install it:

{% codeblock Terminal lang:bash %}
$ bundle
$ rails g acts_as_taggable_on:migration
$ rake db:migrate
{% endcodeblock %}

This will install gem, create and run it's migration to set your database properly. Next, you should attach tags to any existing model of your choice. In my case its the model Movie:

{% codeblock app/models/movie.rb lang:ruby %}
class Movie < ActiveRecord::Base
  acts_as_taggable

  attr_accessible :name, :tag_list
end
{% endcodeblock %}

Interesting thing is that the `acts_as_taggable` is just an alias for `acts_as_taggable_on :tags`. So you can really use any other name for your tags of your own choice. Also make sure that you added `tag_list` in `attr_accessible` section to be able to save tags.

Not it's time to create and then show tags. To do that you should add this simple code inside of your view with the form:

{% codeblock app/views/movies/_form.html.erb lang:erb %}
<div class="field">
  <%= f.label :tag_list, "Your tags (separated by commas)" %><br/>
  <%= f.text_field :tag_list %>
</div>
{% endcodeblock %}

To show tags you can use method `tag_list` which returns array of tags for a current object:

{% codeblock app/views/movies/show.html.erb lang:erb %}
<p>
  <b>Name:</b>
  <%= @movie.name %>
  <br>
  <b>Tags:</b>
  <%= raw @movie.tag_list.join(', ') %>
</p>
{% endcodeblock %}

That should be enough to be able manage your tags with acts_as_taggable_on. Now it's time to implement Jquery token input.

##Jquery token input

To load [jquery-token-input](https://github.com/loopj/jquery-tokeninput) in your app you should add it's js and css files in `vendor/assets` and after that include these files inside of your js and css manifests. For example:

{% codeblock assets/stylesheets/application.css lang:css %}
*= require token-input-mac
{% endcodeblock %}

{% codeblock assets/javascripts/application.js lang:js %}
//= require jquery.tokeninput
{% endcodeblock %}

Please, note, that you css file may have another name (there are few themes available out the box) and it's really okay.

Now I would like to show to you my final coffee's code, which will do the job for my tags:

{% codeblock assets/javascripts/movies.js.coffee lang:coffeescript %}
jQuery ->
  $('#movie_tag_list_tokens').tokenInput '/movies/tags.json',
    theme: 'mac'
    minChars: 2
    allowCustomEntry: true
    preventDuplicates: true
    prePopulate: $('#movie_tag_list_tokens').data('load')
{% endcodeblock %}

This code will grab and show new tags via ajax from the URL `/movies/tags.json`, basing on user input. But really, a tag can't consist of a single symbol, so I set minimum at 2 symbols to prohibit such bad behavior. Jquery token input will validate it for us.

To be able to create a new tag you should use `allowCustomEntry`. Also, a movie can already have tags, so i upload data from the div using data-load tag (you can easily implement it since Rails 3.1) and pass it to the `prePopulate` method. If you still not using Rails 3.1 then you can use instead of that gem [gon](https://github.com/gazay/gon), which I really love. And please, please upgrade Rails to the newer version!

Now it's time to add the method in controller, which will return all founded movies tags (so app will return data on `/movies/tags.json` call). First of all, add route with `tag` method:

{% codeblock config/routes.rb lang:ruby %}
resources :movies do
  collection do
    get :tags, as: :tags
  end
end
{% endcodeblock %}

Now I should add an actual method to the controller:

{% codeblock app/controllers/movies_controller.rb lang:ruby %}
def tags
  tags = Movie.all_tag_counts.by_tag_name(params[:q]).token_input_tags

  respond_to do |format|
    format.json { render json: tags }
  end
end
{% endcodeblock %}

This code will do the job (it returns all movies tags). Note, that I use some tricks on the code above. I want to find tags of specific model (Movie), it makes sense, because I may have many tags for different models, and I want to separate model's tags from each other. That is why I use `all_tag_counts` which returns relation of tags (representing as array) for a `Movie`.

When the app gets all necessary tags via `all_tag_counts` then it should filter them to find tags which’ll match user's input. For that purpose I use the scope `by_tag_name` which find data in received ActsAsTaggableOn’s relation. And finally, you should transform founded tags in the format, which Jquery token input will understand. If you would look up in the [docs](http://loopj.com/jquery-tokeninput/) then you would see that js expects data in the very specific format:

{% codeblock Json tags lang:json %}
[
  {"id":"856","name":"House"},
  {"id":"1035","name":"Desperate Housewives"}
]
{% endcodeblock %}

But when `by_tag_name` scope has fired then I got such relation (as you remember):

{% codeblock by_tag_name output lang:ruby %}
[ #<ActsAsTaggableOn::Tag id: 1, name: "Piter Pan">, #<ActsAsTaggableOn::Tag id: 2, name: "Superman">, #<ActsAsTaggableOn::Tag id: 3, name: "Piter Parker"> ]
{% endcodeblock %}

So I should transform my data for Jquery Token Input and method `token_input_tags` does it.

To implement `by_tag_name` and `token_input_tags` you should define the module in your `lib/` directory and after that include your module in the ActsAsTaggableOn, because the `all_tag_counts` returns ActsAsTaggableOn’s relation.

Here is my module:

{% codeblock lib/extended/tag_extend.rb lang:ruby %}
module TagExtend
  extend ActiveSupport::Concern

  included do
    scope :by_tag_name, -> name { where("name like ?", "%#{name}%") }

    def self.token_input_tags
        scoped.map{|t| {id: t.name, name: t.name }}
    end
  end
end
{% endcodeblock %}

Here I define my scope and method with some help of the ActiveSupport (thank you, guys!). You may be surprised that I put in the id name of a tag, but it's the only workaround which works for ActsAsTaggableOn (otherwise created tag will contain id instead of the name).

Now let's tell Rails to find my libs. In config/application.rb add:

{% codeblock config/application.rb lang:ruby %}
config.autoload_paths += Dir["#{config.root}/lib/**/"]
{% endcodeblock %}

And finally, lets include our module in ActsAsTaggableOn. Create in initializers file `acts_as_taggable_on.rb` and add in that file:

{% codeblock config/initializers/acts_as_taggable_on.rb lang:ruby %}
ActsAsTaggableOn::Tag.send(:include, ActsAsJqueryTokenRails3::TagExtend)
{% endcodeblock %}

ActsAsTaggableOn is now having my scope and the defined method! Awesome!

But I didn't finish. When you create few tags, save your model and then try to edit your model then you expect to see your existing tags. So, lets implement it.

##Prepopulating tags

As you remember, I has already defined such line of the code in my coffee's file:

{% codeblock assets/javascripts/movies.js.coffee lang:coffeescript %}
prePopulate: $('#movie_tag_list_tokens').data('load')
{% endcodeblock %}

To load actual movies tags I should add my data tag in the movie's edit form and then fill it with actual data. Here is updated form:

{% codeblock app/views/movies/_form.html.erb lang:erb %}
<div class="field">
  <%= f.label :tag_list_tokens, "Your tags (separated by commas)" %><br/>
  <%= f.text_field :tag_list_tokens, data: {load: @movie_tags} %>
</div>
{% endcodeblock %}

I didn't just add data but also I changed the name of the field. It's because when you create new tags then you get them in very specific format:

```
"old_tag1,old_tag2,'new_tag1','new_tag2'"
```

As you can see, new tags are wrapped in single quotes. App shouldn't save these quotes, because user hasn't printed them. To do that I define a virtual attribute in my model and override setter to save actual tags without quotes:

{% codeblock app/models/movie.rb lang:ruby %}
class Movie < ActiveRecord::Base
  acts_as_taggable

  attr_reader :tag_list_tokens
  attr_accessible :name, :tag_list_tokens

  def tag_list_tokens=(tokens)
    self.tag_list = tokens.gsub("'", "")
  end
end
{% endcodeblock %}

Please notice, that now you can and should remove `tag_list` from attr_accessible.

All right, it's time to populate our movies tags! For that purpose in movies_controller i added before_filter to call a new method to collect existing tags:

{% codeblock app/controllers/movies_controller.rb lang:ruby %}
before_filter :find_tags, only: [:new, :create, :edit, :update]

#...

private

def find_tags
  @movie_tags = params[:id].present? ? Movie.find(params[:id]).tags.token_input_tags : []
end
{% endcodeblock %}

Now our app works, you can create new tags, add existing one, search them via ajax and even delete them. Thanks for your patience!

P.S. You can find working demo, it uses all these features in [my github's repo](https://github.com/Loremaster/acts-as-jquery-token-rails3).