---
layout: post
title: Memory Purge development (OS X)
---
So, after 4+ years of iOS development I've decided to start developing for OS X. My first app was supposed to be very simple.
I also wanted to try out Swift - new iOS/OS X programming language.

The simpliest project I came up with was RAM cleaning (purging) app. Basicly what app was supposed to do is invoking "purge" terminal command.

Turned out it was not as simple as it seemed to be.

First of all, OS X development is a torture (compared to iOS).
Documentation is outdated, community is small (or better sad tiny), official support/bugreport forum is almost dead, nothing could be found via google/stackoverflow and so on.

Second, the Swift, language that was released before its standart library and even its syntax(!) were stabilized. Many swift examples found now on the internet could not be compiled because of that. Although Swift is a great modern language and probably will completely replace Objective-C one day, it's not production-ready yet. 

Third major issue I faced was 10.9 (Mavericks) OS X update that started requiring super user privileges for "purge" command. So I either had to ask password every time user invokes memory cleaning from my app or implement memory cleaning without calling purge (by means of allocating small memory chunks which extrude cached memory). 
First option was not even considered and second one was declined after a bit of testing. Eventually I came up with third option: modifying sudoers file. That also requires root privs but luckily only once. 
That's actually a very bad practice since breaking sudoers file may cause some [troubles](http://astrails.com/blog/2009/9/29/how-to-fix-a-hosed-etc-sudoers-file-on-mac-osx) but sometimes you just don't have other choice.
The way I update sudoers is probably the safest. Here it is:

1. Copy sudoers to temporary directory
2. Modify the copy
3. Check the copy with "visudo"
4. If and only if visudo gives no error - overwrite the original file with modified

After sudoers file is updated there's no need to enter root password every time the purge is invoked. Hurray! 

Unfortunately this was not the end.
Application also had to display total and free memory and autostart on login.
I won't touch autostart here - it is perfectly described on the internet.
Total and free memory querying on the other hand is not described anywhere at all, so I also won't touch it here :) 
Especially since I'm unhappy with the way I implemented that part.

Ok, just kidding. I'll update this article soon. Stay tuned...
