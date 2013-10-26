---
layout: post
title:  "Enable SSL with Heroku for https access"
date:   2012-07-21 18:59:31
categories:
---

For <a href="https://rstat.us">rstat.us</a>, we've been meaning to enable ssl/https <a href="https://github.com/hotsh/rstat.us/issues/89">for a long time</a>. I'm pleased to announce that we've finally gotten around to it, and everything seems to be working now!

What was the hold up? On heroku (where we started hosting rstat.us, then we weren't, now we are again), it used to cost $100/mo for the ssl hostname addon. I love rstat.us, but not that much. Recently, however, heroku came out with the <a href="https://devcenter.heroku.com/articles/ssl-endpoint">SSL Endpoint</a> addon which is only $20/mo. This is much more within my budget :)

I also wanted to make sure that the Certificate Authority we went with was in line with rstat.us' values. This is a project that started on the values of openness and simplicity, so <a href="http://librelist.com/browser//rstatus/2012/7/10/certificate-authority/">I agonized a bit over our choice of CA</a>. We ended up going with a free certificate from <a href="https://www.startssl.com/">StartSSL</a> for the reasons I mentioned in that thread.

Once you have a certificate, the <a href="https://devcenter.heroku.com/articles/ssl-endpoint">instructions for the SSL Endpoint</a> are pretty good. There are a few places that I either had issues with or think could use some further clarification:

<h2>Intermediate certs</h2>

In the SSL endpoint docs, there's a part that says "If you have uploaded a certificate that was signed by a root authority but you get the message that it is not trusted, then something is wrong with the certificate. For example, it may be missing intermediary certificates." This doesn't say what you need to do to add the intermediary certificates that you may be missing.

Some certificate authorities will have intermediate certificates that have to be included in your .crt file. <a href="https://devcenter.heroku.com/articles/ssl-certificate">The "Purchasing an SSL Certificate" heroku docs</a> explain how to cat your certificate with a root certificate. If you have an intermediate certificate file as well, first cat your certificate as they show in the instructions, then cat the intermediate certificate, then cat the root certificate. I figured this out by trying it and it worked, but I felt unsure since the docs from heroku and the CA didn't explicitly say what order they should go in.

<h2><em>Add</em> vs <em>Edit</em> CNAME</h2>

This may be obvious to everyone but me, but I thought I'd mention it anyway in case someone else has the same brain fart I did. The SSL endpoint docs say "Next, add a CNAME record in the DNS configuration that points from the domain name that will host secure traffic e.g. www.mydomain.com to the SSL endpoint hostname, e.g. tokyo-2121.herokussl.com."

So I did exactly that-- <em>added</em> a CNAME record. But we already HAD a CNAME record for www.rstat.us since we had already followed the <a href="https://devcenter.heroku.com/articles/custom-domains">heroku custom domains instructions</a>. And CNAMES aren't additive, hah. So yeah, if you already have a CNAME for the domain name you're trying to enable ssl with, just <em>edit</em> that one.

<h2>Naked domain</h2>

This is the part I struggled with for a week. I got SSL all working for www.rstat.us according to the SSL endpoint docs-- https://www.rstat.us worked great! I also had a 301 URL Redirect set up in our DNS records with NameCheap to redirect the naked domain rstat.us to www. This may be unpopular, but naked domains can't be CNAMES and using A records for the naked domain is highly discouraged for availability reasons (<a href="https://devcenter.heroku.com/articles/custom-domains">see the naked domain section</a>).

However, 301 redirects don't care about the protocol-- if you go to http://rstat.us, it redirects you to http://www.rstat.us, and if you go to https://rstat.us it (supposedly) redirects you to https://www.rstat.us. What I was seeing, though, if you went to https://rstat.us, was that the SSL handshake would just hang. This was very visible when doing a `curl -v https://rstat.us` on the command line-- it would just get to the SSL handshake and wait forever.

The certificate was good for both rstat.us and www.rstat.us, so that wasn't the problem. It looks like an ordering issue to me (correct me if I'm wrong)-- request https://rstat.us, do the SSL handshake since it's SSL, and THEN do whatever the DNS says to do-- but the CNAME with www that makes SSL work is AFTER the 301 redirect specified in the DNS settings for the naked domain.

<a href="https://devcenter.heroku.com/articles/avoiding-naked-domains-dns-arecords">Heroku's documented solution to this</a> is "only circulating and publicizing the subdomain format of your secure URL." It is left as an exercise for the reader to determine why this is an unacceptable solution.

Not mentioned in that page, but linked in its references, is <a href="http://blog.dnsimple.com/zone-apex-naked-domain-alias-that-works/">a blog post by DNSimple introducing ALIAS records</a>. This is something that, as far as I can tell, only DNSimple has implemented, and it basically turns a CNAME into an A record behind the scenes without any need for user intervention should the IP addresses that the CNAME resolves to change.

<a href="http://stackoverflow.com/questions/6701549/heroku-ssl-on-root-domain">This StackOverflow question</a> confirmed that the DNSimple ALIAS record would solve the problem I was having-- so I set up a DNSimple account ($3/mo for up to 10 domains at the moment, but <a href="https://dnsimple.com/r/c5d138eb58833f">use my referral link and we both get a month free!</a></shameless-rstatus-promotion>), set up the following records:

<pre>
ALIAS 	rstat.us 	www.rstat.us
CNAME 	www.rstat.us 	toyama-8790.herokussl.com
</pre>

and changed the settings with NameCheap, where we have rstat.us registered, to use DNSimple's DNS instead of NameCheap's. Indeed, the ALIAS record magically makes all variations of http/https and naked domain/www work as desired!

Can I have my devops merit badge now?