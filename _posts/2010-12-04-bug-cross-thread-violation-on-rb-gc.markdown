---
layout: post
title:  "[BUG] cross-thread violation on rb_gc()"
date:   2010-12-04 04:44:24
categories:
---

The Problem:

When I ran:

{% highlight ruby %}
&gt; rails generate rspec:install
{% endhighlight %}

or

{% highlight ruby %}
&gt; rails generate cucumber:install
{% endhighlight %}

I got:

{% highlight ruby %}
[BUG] cross-thread violation on rb_gc()
(null)

This application has requested the Runtime to terminate it in an unusual way.
Please contact the application's support team for more information.
{% endhighlight %}

The Solution:

I found <a href="https://groups.google.com/group/rubyinstaller/browse_thread/thread/f46c95810db948be/a70899e15c41a5f8">this post in the RubyInstaller google group</a> where Luis Lavena had worked with someone else seeing the same issue, and he recognized it as a problem with the json gem version 1.4.6 x86-mingw32 not having binaries for Ruby 1.9.

His recommended fix, that worked for me, is to run:

{% highlight ruby %}
&gt; gem uninstall json
{% endhighlight %}

Then <a href="https://github.com/oneclick/rubyinstaller/wiki/Development-Kit">download and install DevKit</a> for the Ruby installation I was using (I'm using pik and was trying to set up a new environment, see the details below)

Then run:
{% highlight ruby %}
&gt; gem install json --platform=ruby
{% endhighlight %}

And then the rails generate rspec:install and rails generate cucumber:install commands worked as expected.

Please comment if this post helped you with the same issue, or if you  know how to fix this issue so that it won't bite others in the future. I'm still a beginning Rubyist (I've gotten  to a point where I'm starting to know how much I don't know), and I'm  definitely interested in working on this and other problems (which was  my original intention of this evening!!!) but I don't know exactly how  to go about doing such things yet, in many cases.

More details:

I'm using pik, the Windows alternative to rvm. My overall goal was to create a new, clean pik environment in which I could create a new rails app using a clean gemset.

So I used the <a href="http://rubyinstaller.org/">RubyInstaller</a> (thank you again, Luis!) to install Ruby 1.9.2, then I added it to pik and switched to it.

Then I started installing-- rails 3.0.3 and cucumber 0.9.4 (the gem I'm interested in reproducing a bug with/patching).

At that point I switched to following <a href="http://pragprog.com/titles/achbd/the-rspec-book">The RSpec Book</a> starting on page 286 (of printing 1) which has you do:

{% highlight ruby %}
&gt; rails new &lt;my-app&gt;
&gt; cd &lt;my-app&gt;
{% endhighlight %}

Modify your Gemfile to be:

{% highlight ruby %}
source 'http://rubygems.org'

gem 'rails', '3.0.3'
gem 'sqlite3-ruby', :require =&gt; 'sqlite3'

group :development, :test do
gem 'rspec-rails', "&gt;=2.0.0"
gem 'cucumber-rails', "&gt;=0.3.2"
gem 'webrat', "&gt;= 0.7.2"
end
{% endhighlight %}

Then run:

{% highlight ruby %}
&gt; bundle install
{% endhighlight %}

And then finally:

{% highlight ruby %}
&gt; rails generate rspec:install
{% endhighlight %}

And that is how I got to the cross-thread violation on rb_gc(). Phew.

How I got to the solution:

Google searches for:
<ul>
	<li>rails generate rspec install bug cross-thread violation</li>
	<li>"cross-thread violation on rb_gc"</li>
	<li>pik ruby windows cross-thread violation rb_gc</li>
	<li>+pik +ruby +windows +cross-thread +violation +rb_gc (yes, google, i really want ALL these terms to be in the pages you give me... what is this, 1997? That's another post...)</li>
	<li>site:ruby-forum.com "cross-thread violation on rb_gc"</li>
	<li>bug cross-thread violation dll (i had seen a dll having to do with 1.8 being referenced in the details of the crash as reported by windows)</li>
	<li>bug cross-thread violation rb_gc dll</li>
</ul>
Stack overflow searches for:
<ul>
	<li>[ruby] [cucumber] cross-thread violation</li>
	<li>[cucumber] cross-thread violation</li>
	<li>[rspec] cross-thread violation</li>
</ul>
I also looked at the issue trackers for pik and a lot of posts on the Ruby Forum.

I have now learned that the RubyInstaller google group is a good place to start looking for windows+ruby related issues.