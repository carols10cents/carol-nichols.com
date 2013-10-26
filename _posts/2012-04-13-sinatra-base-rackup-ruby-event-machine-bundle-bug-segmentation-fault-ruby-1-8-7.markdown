---
layout: post
title:  "Sinatra:Base rackup: rubyeventmachine.bundle: [BUG] Segmentation fault ruby 1.8.7"
date:   2012-04-13 19:08:12
categories:
---

I was *just* saying yesterday that I haven't run into any strange errors lately. I guess I forgot to knock on wood!

Today I started a brand new sinatra app. I haven't written my own sinatra app from scratch before, so I'm copying pieces from some other apps that I have cloned on my machine. Here's what my code looked like:

{% highlight ruby %}
# Gemfile
source :rubygems

gem 'sinatra'
gem 'thin'
{% endhighlight %}

{% highlight ruby %}
# salmon_test.rb
require 'sinatra'

class SalmonTest < Sinatra::Base
  get "/" do
    "hello"
  end
end
{% endhighlight %}

{% highlight ruby %}
# config.ru
require './salmon_test'
run SalmonTest
{% endhighlight %}

Then I bundled and ended up with this Gemfile.lock:
{% highlight ruby %}
GEM
  remote: http://rubygems.org/
  specs:
    daemons (1.1.8)
    eventmachine (0.12.10)
    rack (1.4.1)
    rack-protection (1.2.0)
      rack
    sinatra (1.3.2)
      rack (~> 1.3, >= 1.3.6)
      rack-protection (~> 1.2)
      tilt (~> 1.3, >= 1.3.3)
    thin (1.3.1)
      daemons (>= 1.0.9)
      eventmachine (>= 0.12.6)
      rack (>= 1.0.0)
    tilt (1.3.3)

PLATFORMS
  ruby

DEPENDENCIES
  sinatra
  thin
{% endhighlight %}

Also note that I'm using rvm with ruby 1.9.2-p290 and a brand new gemset for this project.

When I ran <code>rackup</code> to start the server, I got this error message:

{% highlight shell %}
$ rackup
~/.rvm/gems/ruby-1.9.2-p290@salmon_test/gems/eventmachine-0.12.10/lib/rubyeventmachine.bundle: [BUG] Segmentation fault
ruby 1.8.7 (2010-01-10 patchlevel 249) [universal-darwin10.0]

Abort trap
{% endhighlight %}

Why is it doing something with 1.8.7??? Who knows! Right after that I did an <code>rvm list</code>:

{% highlight shell %}
$ rvm list

rvm rubies

   ruby-1.8.7-p358 [ i686 ]
=* ruby-1.9.2-p290 [ x86_64 ]
   ruby-1.9.2-p318 [ x86_64 ]
   ruby-1.9.3-p125 [ x86_64 ]
{% endhighlight %}

Yep, using 1.9.2...

The one thing I changed in the code before running <code>rackup</code> again was in salmon_test.rb:

{% highlight diff %}
- require 'sinatra'
+ require 'sinatra/base'
{% endhighlight %}

Then the next time I ran rackup everything worked fine. *shrug* Hope this helps someone else.