The json gem strikes again: heroku on Windows

This json gem without windows binaries for Ruby 1.9.2 is the bane of my existence.

I just started using heroku today with the heroku gem, and on every command I got the dreaded "msvcrt-ruby18.dll was not found" popup.

As usual, <a href="http://stackoverflow.com/questions/2167992/problem-with-ruby-on-rails-on-windowsmsvcrt-ruby18-dll-error-newbie-questions">the fix is to reinstall the json gem</a> (after <a href="https://github.com/oneclick/rubyinstaller/wiki/Development-Kit">getting DevKit</a>):

<strong>Edit:</strong> As pointed out by <a href="http://carol-nichols.com/2011/02/the-json-gem-strikes-again-heroku-on-windows/#comment-112">Chad Tipton in the comments</a>, you'll need to add --version=1.4.6 because 1.5.1 is out but 1.4.6 is the version of the json gem that the heroku gem depends on.

{% highlight ruby %}
&gt; gem uninstall json
&gt; gem install json --platform=ruby --version=1.4.6
{% endhighlight %}

I also learned that there's a batch script to set your PATH for using DevKit in your DevKit install dir called devkitvars.bat that will add install-dir\bin to your PATH... that's what I get for messing around with my PATH for no good reason.

I really should just start running "gem install json --platform=ruby" after every command.