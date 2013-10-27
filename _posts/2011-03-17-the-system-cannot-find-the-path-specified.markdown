---
layout: post
title:  "The system cannot find the path specified"
date:   2011-03-17 19:40:09
categories:
---

So recently I started getting many incidents of the error "The system cannot find the path specified" when running just about anything from the windows command line and sometimes from the Git Bash bash. It was really weird because everything seemed to be working just fine: tests would pass, rails would function normally, etc, but I would get "The system cannot find the path specified" between various lines of output with seemingly no rhyme or reason.

I let this error go for a while since it wasn't really breaking anything, but eventually it started to piss me off because it wasn't even letting me know WHAT path it couldn't find!

And to make matters worse, around the same time that the error started happening I had:

<ul class="bulleted-list">
  <li>Installed JRuby on this computer for the first time</li>
  <li>Moved the physical location of the computer and was plugged into a different ethernet port</li>
  <li>Mucked about with my PATH and some other environment variables</li>
  <li>Tried to organize some files in a saner way</li>
</ul>

I really had no idea which one of these things was likely responsible. Searching for this problem was tough since the error message was so general and can be caused by any number of things, and I didn't find anything that corresponded to the things I had done above, exactly.

So eventually I asked <a href="http://jakegoulding.posterous.com/">Jake</a> what to do about this, and he recommended using <a href="http://technet.microsoft.com/en-us/sysinternals/bb896645">Process Monitor</a> to try and see what path was not being found. I fired it up, set up a filter to only show events from cmd.exe, ran a fast-running command that I knew would have the error, and started reading the logs.

The key entry had a Result of "PATH NOT FOUND":

<table class="data">
  <tr>
    <th>Process Name</th>
    <th>Operation</th>
    <th>Path</th>
    <th>Result</th>
    <th>Detail</th>
  </tr>
  <tr>
    <td>cmd.exe</td>
    <td>CreateFile</td>
    <td>C:\Documents and Settings\nichols\Desktop\ansi132\x86\</td>
    <td>PATH NOT FOUND</td>
    <td>Desired Access: Read Data/List Directory, Synchronize, Disposition: Open, Options: Directory, Synchronous IO Non-Alert, Attributes: n/a, ShareMode: Read, Write, AllocationSize: n/a</td>
  </tr>
</table>

And there we have the culprit: me, surprise surprise ;) When the standard way of getting terminal colors on windows switched from the windows32 gem to ANSICON (<a href="https://groups.google.com/group/rubyinstaller/browse_thread/thread/2d2a62db7281509a/19bac4baa8c3845d?lnk=gst&q=ansi#">in RubyInstaller</a>, <a href="https://github.com/aslakhellesoy/cucumber/commit/1a2bd170ef7b292031a5e32fe77e8795825f4820">Cucumber</a> and <a href="https://github.com/rspec/rspec-core/commit/d5a39c92f90bc21ae6fe2ed54429033a680572bd">RSpec</a>), the first time I had installed <a href="http://adoxa.110mb.com/ansicon/">ANSICON</a> (by running it with -i) was from that location on my desktop. Then I moved it when doing the cleanup mentioned above, and I didn't run -u before I moved it, so the <a href="https://github.com/adoxa/ansicon">registry entry that -i adds</a> didn't get removed.

The particular registry entry causing the trouble was a few lines up in the Process Monitor logs; it was "HKCU\Software\Microsoft\Command Processor\AutoRun". It actually had both the old location and the new location of ANSICON at this point, so I backed up my registry to be safe, removed the path to my Desktop from that value, and the errors were no more! Whew.