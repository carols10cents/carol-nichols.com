Updating gems in rvm's global and default gemsets

This is mostly for my own reference; I know I've looked this up before and it's always hard to find, so at least now I'll have one place to look.

If you want to update gems that you have in your default or global gemsets because you see 2 versions of, say, bundler when you do gem list, you can install and uninstall from them by using a command like this (including the parentheses):

<code>(rvm use @global; gem uninstall -x bundler)</code>

Switching global and default as necessary, and uninstall and install, and bundler and rake (the 2 gems I get in this situation with).

UPDATE: Just talked with Michal Papis (<a href="https://twitter.com/#!/mpapis">@mpapis</a>) in IRC and he recommends using:

<code>rvm @global do gem uninstall -x bundler</code>

which looks much nicer. As an aside, I was having an issue yesterday and Michal fixed it by the time I got into work today! Thank you so much for your help and work on rvm, Michal!!