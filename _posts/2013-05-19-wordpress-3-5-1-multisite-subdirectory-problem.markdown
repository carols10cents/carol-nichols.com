Wordpress 3.5.1 multisite subdirectory problem

I recently created a new wordpress 3.5.1 install with the intention of enabling multisite. I followed all the <a href="http://codex.wordpress.org/Create_A_Network">official instructions</a> and everything seemed to be working until I got to actually creating a second site in the network with a subdirectory.

I created the site (ex: named 'blah'), but when I tried to go to domain.com/blah or domain.com/blah/wp-admin, it was like I was looking at the main site in the network.

I tried many different recommended .htaccess variations, checked every gotcha that official wordpress documentation, <a href="http://wordpress.stackexchange.com">the wordpress stackexchange</a>, and other wordpress blogs mentioned, to no avail.

Finally it took some help from my life pair <a href="http://jakegoulding.com/">Jake Goulding</a> to spot that in the site info for the subsite, I had '/blah' for the path setting, and since the main site had '/' as the path, perhaps it should be '/blah/'.

Adding the trailing slash in the path for the site allowed wordpress to recognize it and going to domain.com/blah/wp-admin immediately started working.

I admittedly am not very experienced with wordpress, so perhaps this is something everyone else was able to figure out, especially since I didn't see anyone else recommending that you check this.

But I found the help text in the Add a Site form to be misleading:

<a href="http://carol-nichols.com/wp-content/uploads/2013/05/Screen-Shot-2013-05-19-at-5.39.03-PM.png"><img src="http://carol-nichols.com/wp-content/uploads/2013/05/Screen-Shot-2013-05-19-at-5.39.03-PM.png" alt="" title="Screen Shot 2013-05-19 at 5.39.03 PM" width="560" class="aligncenter size-full wp-image-192" style="max-width: 560px;" /></a>

It says "Only lowercase letters (a-z) and numbers are allowed." To me, that says "You should not enter a trailing slash", when in fact the behavior I see is that you MUST enter a trailing slash. Either the help text should be changed or, even better, this should work whether you enter a trailing slash or not.