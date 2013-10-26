---
layout: post
title:  "cucumber + win32console, continued"
date:   2010-12-05 23:11:30
categories:
---

So [since I didn't have too much luck last night](/2010/12/04/lets-try-this-again-shall-we-win32console-problems) in getting win32console installed and Cucumber to recognize that I had win32console installed, I decided to try looking for the other problem I was seeing: "uninitialized constant win32".

Googling that error message got me to <a href="http://doelsengupta.blogspot.com/2010/10/uninitialized-constant-win32-nameerror.html">this post</a> which reminded me that I'm not working in Rails 2.3.x (like I do at my day job). So then I added win32console to my development group in my gemfile, did a bundle install, and rake cucumber then got me the 0 scenarios, 0 steps output without any errors.

While I was waiting for the rake cucumber to finish though (performance on windows is another thing I'd like to work on), I read through a few more results and saw that there's <a href="https://rspec.lighthouseapp.com/projects/16211/tickets/358-color-loadingwarning-ignores-no-color">a related bug in cucumber</a>: this error happens on windows with ruby 1.9.2 even if you tell cucumber to not use color. Maybe that will be the next bug I work on.
