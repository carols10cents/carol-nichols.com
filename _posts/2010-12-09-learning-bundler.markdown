---
layout: post
title:  "Learning Bundler"
date:   2010-12-09 17:37:15
categories:
---

So after I figured out how to fix [the cross-thread violations](/2010/12/03/bug-cross-thread-violation-on-rb-gc) due to the json gem, and the other problems I ran into, I reproduced <a href="https://rspec.lighthouseapp.com/projects/16211/tickets/489-rails-model-naming-conflict-with-term#ticket-489-4">the bug I was actually attempting to fix</a>. Then I checked out cucumber, changed my Gemfile to use my dev version of cucumber, did a bundle install and started poking around in the code while that finished. I noticed that the newest version of cucumber now uses <a href="http://adoxa.110mb.com/ansicon/">ANSICON</a> to do colors on windows-- I'm not yet sure how that affects the bug I'm trying to fix that has to do with term-ansicolor.

One of the reasons I'm not yet sure is that, when I tried to reproduce the bug I'm trying to fix again once bundle install was done, the cross-thread violation was back! Since I didn't specify what json gem I wanted at all in my Gemfile, cucumber pulled in the bad version of the json gem again.

<s>I also learned that I needed to put my json specification:</s>

{% highlight ruby %}
  gem 'json', '=1.4.6', :platforms => [:ruby]
{% endhighlight %}

<s>BEFORE my cucumber specification, in order to be sure that cucumber didn't dictate the version of json to be used.</s>

Nevermind, it seems to be randomly choosing whether to use the json that cucumber wants or the version I have on my system... about half the time when I run bundle install I have to uninstall the mingw32 json gem. Sigh.

While figuring this out, I read <a href="https://github.com/carlhuda/bundler/issues/labels/windows#issue/635">this bundler bug</a> wherein the Gemfile.lock (which you are supposed to check into your vcs) is generated differently on windows and linux/mac. The comments in that bug also show the subtle solution you'd have to use in your Gemfile to get the patched version of Nokogiri that is windows-only to work on all platforms:

{% highlight ruby %}
gem "nokogiri", "~> 1.4.4"
{% endhighlight %}

so that you'll get the latest version with bugfixes available on the platform you're on.

Someone suggested being able to specify multiple versions of the same gem depending on the platform, and a bundler dev said that that goes against bundler's reason for existence. I see that point, but saying something like (pseudocode) :windows => { gem "nokogiri", "=1.4.4.1"} [:linux, :mac] => { gem "nokogiri", "=1.4.4" } seems a lot more explicit and makes your app's needs clearer, in the sense of conveying the reasoning behind that choice to future developers/users. I don't know, hard problems are hard.

That bug says a solution for it is targeted for bundler 1.1 and that they are working on improving windows support, so that's good.
