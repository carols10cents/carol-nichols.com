---
layout: post
title:  "attach_file with cucumber+capybara+selenium on Windows"
date:   2011-01-12 14:16:01
categories:
---

Yesterday I was working with some scenarios having to do with file upload, and I thought they were failing because of something I was doing in the code, but they were not failing for my colleague on a mac (you don't have to tell me what the obvious solution to this is, I know).

When I used the attach_file method in capybara with the selenium driver, the file input field would get filled in, but nothing would get submitted-- there just wouldn't be anything for the file parameter in the logs. What's weird is that file upload worked just fine if I did it manually.

I checked out the capybara code and ran the gem's specs on my machine, and sure enough, the specs testing attach_file with selenium failed for me as well.

The issue seems to be with the direction of the slashes in the paths. When I used attach_file, the paths would look like:

{% highlight bash %}
C:/Ruby/capybara/lib/capybara/spec/fixtures/test_file.txt
{% endhighlight %}

While doing it manually through the browser gave me paths that looked like:

{% highlight bash %}
C:\Ruby\capybara\lib\capybara\spec\fixtures\test_file.txt
{% endhighlight %}


So if I changed my step definition in features/step_definitions/web_steps.rb for attaching a file to be:

{% highlight ruby %}
When /^(?:|I )attach the file "([^\"]*)" to "([^\"]*)"(?: within "([^\"]*)")?$/ do |filename, field, selector|
  with_scope(selector) do
    filepath = File.join(RAILS_ROOT, 'features', 'upload_files', filename)
    if filepath.match(/^C:\//)
      filepath.gsub!(/\//, "\\")
    end
    attach_file(field, filepath)
  end
end
{% endhighlight %}

then my cucumber scenarios worked as expected.

I've <a href="">posted to the capybara mailing list</a> to see if anyone knows where this problem is originating (and where a patch should be made) and if this is a good solution or if it's just getting around a more fundamental problem. Anyone reading this have any ideas?