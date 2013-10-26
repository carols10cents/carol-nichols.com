---
layout: post
title:  "Bundler seems to be helping to prevent name conflicts"
date:   2010-12-11 00:10:45
categories:
---

So I'm not totally sure because I'm (obviously) not all that familiar with bundler, but it seems that if I have done a bundle install with a Gemfile where cucumber (and thus term-ansicolor) is included, "rails generate model Term" fails and says:

{% highlight ruby %}
The name 'Term' is either already used in your application or reserved by Ruby on Rails. Please choose an alternative and run this generator again.
{% endhighlight %}

I'm assuming this has to do with the stuff bundler does to make sure there aren't gem conflicts? I'm not sure how to look up if this is something bundler is doing...

Bu I did some experimentation-- if I remove cucumber from the Gemfile, rails generate lets me create a model named Term just fine. Checking with a rails 2.3.8 app that doesn't use bundler but in an environment where cucumber is installed DOES let you generate a model named Term-- I checked that just to be sure I wasn't imagining the bug I saw.

So you can still hit the bug if you create your model called Term, then add cucumber to your Gemfile, then run bundle install, then run rake cucumber.

This is definitely an improvement-- I mostly wanted to fix this bug for people who, like me, were starting out not knowing much but knowing that they wanted to use cucumber and try and do BDD in the "right" way advocated by The RSpec Book, etc. and just happened to want to name their model Term. As long as they install Cucumber first, they should get a nice error message that tells them up front that they have to use a different model name, rather than a cryptic error later.

Now I have to decide if the bug is still worth me fixing... hmm.