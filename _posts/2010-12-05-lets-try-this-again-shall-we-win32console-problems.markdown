---
layout: post
title:  "Let’s try this again, shall we? win32console problems"
date:   2010-12-05 00:37:22
categories:
---

So after <a href="http://carol-nichols.com/?p=11">last night's attempt</a> to get a working windows+ruby+rails+cucumber environment and the blogging thereof, I was ready to make some real progress tonight.

No dice. I got through a handful more commands in the RSpec book before I hit another issue.

I got the rails generate rspec:install and rails generate cucumber:install to work last night. The next steps, on pg 288 (of printing 1), were these, to make sure everything was wired up correctly:

{% highlight ruby %}
&gt; rake db:migrate # worked fine

&gt; rake db:test:prepare # worked fine

&gt; rake spec # got the expected output about not having any *_spec.rb files

&gt; rake cucumber # :(
{% endhighlight %}

I got 2 problems from this command: 'You must "gem install win32console"' and 'uninitialized constant Win32 (NameError)'. <a href="https://gist.github.com/728820">Here's a gist of the whole output</a>.

I decided to ignore the uninitialized constant error for the moment and go install win32 console. I ran the command exactly as instructed, and saw this go by:

{% highlight ruby %}
> gem install win32console
Temporarily enhancing PATH to include DevKit...
Successfully installed win32console-1.3.0-x86-mingw32
1 gem installed
Installing ri documentation for win32console-1.3.0-x86-mingw32...
Installing RDoc documentation for win32console-1.3.0-x86-mingw32...
{% endhighlight %}

And again:

{% highlight ruby %}
> rake cucumber
...
*** WARNING: You must "gem install win32console" (1.2.0 or higher) to get coloured output on MRI/Windows
...
{% endhighlight %}

But I already did that!!! Why isn't cucumber recognizing that I installed win32console?!?

I started by googling for:
<ul>
<li>i installed win32console but cucumber still</li>
</ul>
Remembering the lesson I just added to the previous post, I then searched the RubyInstaller google group for win32console. I found some talk about <a href="https://groups.google.com/group/rubyinstaller/browse_thread/thread/2d2a62db7281509a">deprecating win32console</a>.

That's an interesting discussion for the future that I'm going to look into eventually. For now, I also found <a href="http://www.ruby-forum.com/topic/205569">this ruby-forum post by Luis</a> talking about a prerelease version of win32console. So I did gem uninstall win32console and gem install win32console --prerelease, then did rake cucumber again:

*** WARNING: You must "gem install win32console" (1.2.0 or higher) to get coloured output on MRI/Windows.

And my eyes are drooping faster than my fingers can google, so I'm going to get some rest. I'll be back.
