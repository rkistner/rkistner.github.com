---
layout: post
title: "Single-file Rails 3.2 Application"
description: "Creating a Rails 3.2 application in a single file for testing a gem."
category: ruby
excerpt: "Creating a Rails 3.2 application in a single file for testing a gem."
tags: [ruby, rails, rspec]
---
{% include JB/setup %}

Creating a Rails 3.2 application in a single file for testing a gem.

*Update 2013/05/27:* configure Rails.root for the application.

## Background

Gems often provide functionality specific to Rails applications, without necessarily depending on Rails themselves.
To properly test these gems, we need a Rails application using it. I often see gems like this including the entire
project structure for a dummy Rails application in their spec or test folder.

In the past there have been some gems such as [plugin_test_helper](https://github.com/pluginaweek/plugin_test_helper)
 and [combustion](https://github.com/pat/combustion) that reduces the number of boilerplate files you need in your
 dummy application. However, I recently found that even without helper gems like these, you can define your
 entire Rails 3.x application in a single file.

## Example Rails application

I based this example off this [blog post](http://julio-ody.tumblr.com/post/596997601/small-and-sexy) and
 [example](http://forrst.com/posts/Tiny_Rails_3_working-egK) by Julio Cesar Ody, and adapted it slightly to work
 in Rails 3.2.

Below is an example I use for testing middleware. I place this file in `spec/dummy/application.rb`.

{% highlight ruby %}
require 'rails'
require 'action_controller/railtie'

class Dummy < Rails::Application
  # Set Rails.root to the same folder as this file
  config.root = File.dirname(__FILE__)

  # Rails needs these keys, but they don't really have to be secret for our tests
  config.session_store :cookie_store, key: '****************************************'
  config.secret_token = '****************************************'

  # Log to spec/dummy/test.log
  config.logger = Logger.new(File.expand_path('../test.log', __FILE__))
  # This is important, otherwise the tests will fail
  Rails.logger = config.logger

  # The middleware I'm testing
  config.middleware.use MyMiddleware

  # Our routes
  routes.draw do
    get '/'  => 'dummy#index'
    get '/other' => 'dummy#other'
  end
end

# A simple controller
class DummyController < ActionController::Base
  def index
    render text: 'Home'
  end

  def other
    render text: 'Other'
  end
end
{% endhighlight %}

## Using the application in tests/specs

For my tests I'm using rack/test. Using the application is as simple as requiring the application file, and telling
rack/test to use the it:

{% highlight ruby %}
require 'rspec'
require 'rack/test'

# Configure the Rails application
ENV['RAILS_ENV'] = 'test'
require 'dummy/application'

describe "rails specs" do
  # Tell rack/test to use our Rails dummy application
  let(:app) do
    Rails.application
  end

  it "should get the index page" do
    get '/'
    last_response.status.should == 200
    last_response.body.should =~ /Home/
  end
end
{% endhighlight %}

That's it! For a more complete example, check out my
[simple_admin_auth](https://github.com/embarkmobile/simple_admin_auth/tree/master/spec) gem.
