---
layout: post
title:  "And we're back"
date:   2013-10-26 18:46:00
categories:
---

Sigh. My wordpress blog apparently got hacked, my host shut it down, so I've
spent the last few days converting this blog to a Jekyll site hosted on github
pages. I really can't recommend wordpress to anyone anymore, given the massive
botnets that are attacking every wordpress site out there right now. For
non-technical folks or lazy technical people who don't want to have to spend
time keeping up with updates or fighting off attacks, it's just not a viable
solution.

So here we are. I still have a lot of work to do... the links between posts are
all broken, I had some google juice on some of the posts at least, so it'd be
nice if links to my old posts still redirected. I haven't decided whether to
add commenting via disqus or similar-- I like hearing that my posts have helped
people, but do I really need them? They're really just another avenue for spam.
I'm also not wild about this theme, it's nice enough, but it's just not really
*me* (but I also don't have the time/talent to make something I like better).

Oh, and it's a good thing I'm not very prolific because I ended up converting
my posts by hand. I had a sql dump file, not a wordpress xml export, and
apparently the jekyll-import gem only supports connecting to a database or
importing from the wp xml file. [And I had enough problems just getting the
jekyll import process to tell me
that](https://github.com/mojombo/jekyll/pull/1662) that I didn't exactly have
much confidence that the import process was going to go well anyway.

Technology sucks. Everything's broken.