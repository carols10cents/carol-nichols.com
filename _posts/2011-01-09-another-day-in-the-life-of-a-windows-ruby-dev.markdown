---
layout: post
title:  "Another day in the life of a Windows+Ruby dev"
date:   2011-01-09 17:41:10
categories:
---

So today I decided to try to fix <a href="https://rspec.lighthouseapp.com/projects/16211/tickets/358-color-loadingwarning-ignores-no-color">issue 358 of cucumber</a>. I'm not even to the point of reproducing the bug yet. *sigh*

So I checked out the cucumber source and tried to install it, at which point rake decided to stop working since cucumber has rake as a development dependency and <a href="http://www.lostechies.com/blogs/derickbailey/archive/2010/10/11/ruby-v1-9-2-on-windows-can-t-find-executable-rake-for-rake-0-8-7.aspx">trying to install rake on windows with 1.9.2 breaks rake</a>. Luckily, there was a quick workaround for that (deleting rake.gemspec).

*Then*, since I was trying to do <code>rake install\_gem</code> (which is the [documented way to install cucumber from source](https://github.com/aslakhellesoy/cucumber/wiki/Install) and appears to be a Bundler task), I ran into the bug described in [this ruby-bundler google group post](http://groups.google.com/group/ruby-bundler/browse_thread/thread/96152d7357745bb0.) wherein that task would quit with error message "Unable to determine name from existing gemspec. Use :name => 'gemname' in #install\_tasks to manually set it." (and no, following the instructions in the error message doesn't help). Arve in that post seems to have a patch to bundler to fix it, which I tried applying, but it didn't fix it for me.

So then I just decided to try <code>rake -T</code> in cucumber, and there are tasks <code>build</code> and <code>install</code> that work and also seem to be Bundler tasks, even though install_gem doesn't work.

I think I can finally try reproducing that cucumber bug now, and I'll put looking into the bundler issue on my list for another day.

At least I'm learning a lot!