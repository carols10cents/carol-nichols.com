Google Chrome AJAX POST request: net_error = -100 (CONNECTION_CLOSED)

I haven't totally figured this out yet. I'm probably doing something dumb.

I have a rails application with an action that I'm hitting via an AJAX POST request. This action generates a PDF and saves it on the server, then sends an email using an ActionMailer, then responds with either a success message or an error message.

Everything was working just hunky-dory except in Google Chrome. In Chrome, at first, it looked like the AJAX request wasn't even being fired because I wasn't seeing it in the console like I'm used to in Firebug's console. Eventually I found where it was hiding-- in the Net tab :P

There it didn't look like there were any errors except that in the Status column it said "(canceled)". WTF?!? I wasn't canceling it, the rails server said it was handling it and returning the response I was sending just like normal, so what was going on???

By clicking on the request in the Net tab, or by opening chrome://net-internals/ in a new tab and going to the Events tab, I finally noticed this in the response headers:

{% highlight shell %}
    Content-Transfer-Encoding: binary
    Content-Type: text/html
    Content-Disposition: inline; filename="pdf_141.pdf"
    Content-Length: 5274
{% endhighlight %}

The headers from the PDF generation are bleeding out into my AJAX response for some reason!! I'm still not sure why this is happening. Then I think Chrome sees that the actual data doesn't match the headers and just stops processing.

Once I changed my rails code from:

{% highlight ruby %}
    render :text => "OK"
{% endhighlight %}

to:

{% highlight ruby %}
    send_data "OK", :type => "text/plain"
{% endhighlight %}

<em>aside: yes, i know <a href="https://twitter.com/#!/garybernhardt/status/123892778622132224">i fail REST forever</a></em>

Then the AJAX request happened as I expected it to, and the headers had Content-type: text/plain. *sigh*.