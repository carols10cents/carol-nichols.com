Selenium::WebDriver::Error::UnhandledError (NS_ERROR_ILLEGAL_VALUE)

Somewhere around the time that I got a Mac, upgraded Firefox, upgraded selenium-webdriver, upgraded Capybara and started running lots of acceptance tests in selenium, I started getting errors like this:

{% highlight shell %}
Selenium::WebDriver::Error::UnhandledError: Component returned failure code: 0x80070057 (NS_ERROR_ILLEGAL_VALUE) [nsIDOMXPathEvaluator.createNSResolver]
{% endhighlight %}

They don't appear to correlate with anything particular that my tests are doing-- I can run the same tests again immediately and have different tests get this error, or have no tests get this error at all. Like all good <a href="https://secure.wikimedia.org/wikipedia/en/wiki/Unusual_software_bug#Heisenbug">Heisenbugs</a>, it was especially hard to trigger when I sat down to dig into this!!!

I did the usual googling and the most helpful thing I found was <a href="https://groups.google.com/group/ruby-capybara/browse_thread/thread/1500b9a7a79877e2?pli=1">this Capybara mailing list post</a> explaining different ways of debugging Selenium UnhandledErrors.

I didn't see anything initially useful in the output when running with $DEBUG on.

Dumping the firefox console log to disk (<a href="http://www.allenwei.cn/tips-add-firebug-extension-to-capybara/">this post was clearer to me than the capybara docs on how to do this</a>) showed this error that looked like what I saw:

{% highlight shell %}
nsCommandProcessor.js:314 - Exception caught by driver: findElements([object Object])
[Exception... "Component returned failure code: 0x80070057 (NS_ERROR_ILLEGAL_VALUE) [nsIDOMXPathEvaluator.createNSResolver]"  nsresult: "0x80070057 (NS_ERROR_ILLEGAL_VALUE)"  location: "JS frame :: resource://fxdriver/modules/atoms.js :: <TOP_LEVEL> :: line 2354"  data: no]
{% endhighlight %}

So then I searched for "firefox nsCommandProcessor.js:314 - Exception caught by driver" and found <a href="https://code.google.com/p/selenium/issues/detail?id=2099">this selenium issue</a>. Which says it's a timing issue in Selenium between get and findElement. I don't know right now if there's a way to sleep in my tests to solve it.

And now i've hit a timing issue that I know sleep will cure :D Good night-- I might look into a workaround more tomorrow.

My system:
<ul>
  <li>OSX 10.6.8 (no, I haven't upgraded to Lion yet :P)</li>
  <li>Ruby 1.8.7-p302</li>
  <li>Firefox 5.0.1</li>
  <li>selenium-webdriver 2.0.1</li>
  <li>capybara at <a href="https://github.com/jnicklas/capybara/commit/4fc07dbdc8146bb7ea6039779af6b0fef2196abb">4fc07dbdc814</a></li>
</ul>

<em>UPDATE Jul 26</em>: There have been a few updates to <a href="https://code.google.com/p/selenium/issues/detail?id=2099">the bug</a> today, with some speculation about the cause of the error. One commenter says that they're seeing the problem with post-redirect-gets, but that's not what I'm experiencing-- I'm getting the problem most with regular old links. The same commenter is suggesting a race condition, so I'm trying adding a <code>sleep 1</code> after every line in my script that triggers the error-- after the click_link, before the next statement (that capybara automatically is doing a wait/find on). This seemed to help at first, but just kept happening at different places instead. I also tried adding a sleep 1 in <a href="https://github.com/jnicklas/capybara/blob/master/lib/capybara/node/actions.rb#L25">capybara's click_link</a> and that did not fix the issue.

BUT! If I add a <code>sleep 1</code> to be the first line in capybara's selenium find in <a href="https://github.com/jnicklas/capybara/blob/master/lib/capybara/selenium/driver.rb#L48">lib/capybara/selenium/driver.rb</a>, this *seems* to be enough to get around the race condition (knock on wood). I've run tests that have consistently hit the error a few times now without hitting it, so...

To be clear:
The workaround I've had good results with is changing def find in lib/capybara/selenium/driver.rb to be:
{% highlight ruby %}
   def find(selector)
    sleep 1
     browser.find_elements(:xpath, selector).map { |node| Capybara::Selenium::Node.new(self, node) }
   end
{% endhighlight %}

Let me know if that works for you. Obviously this is going to make your tests slower, but since you're running Selenium tests already I'm assuming you aren't <a href="https://twitter.com/#!/garybernhardt">Gary Bernhardt</a> and can put up with slow but passing tests for a while.

And this is a workaround, the bug is with Selenium, NOT with capybara. The real solution will be a fix to <a href="https://code.google.com/p/selenium/issues/detail?id=2099">this bug</a>, so if you are hitting this, go star that issue.