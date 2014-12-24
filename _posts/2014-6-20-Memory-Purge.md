---
layout: post
title: Memory Purge development
---
So i've decided to start developing for OS X. My first app was supposed to be very simple.
I also wanted to try out Swift - new iOS/OS X programming language.
The simpliest project I came up with was operatve memory cleaning (purging) app. Basicly what app was supposed to do is invoking "purge" terminal command.

Turned out it was not as simple as it seemed to be.
First of all, OS X development is a torture (compared to iOS).
Documentation is outdated, community is small (or better sad tiny), official support/bugreport forum is almost dead, nothing could be found via google/stackoverflow and so on.

Second, the Swift, language that was released before standart library and even its syntax(!) rules were stabilized. Many swift examples found now on the internet could not be compiled because of that. Although Swift is a great modern language and probably will completely replace Objective-C one day, it's not production-ready yet. 

Third major issue I faced was 10.9 (Mavericks) OS X update that started requiring super user privileges for "purge" command. So I either had to ask password every time user invokes memory cleaning from my app or try implementing memory cleaning without calling purge. First option was not even considered and second one was declined after a bit of testing. After some time I came up with third option: modifying sudoers file. That also requires root privs but luckily only once. That's actually a very bad practive since breaking sudoers file may cause some troubles but sometimes you just don't have other choice.
The way I update sudoers is probably the safest. Here it is:

Copy sudoers to temporary directory
Modify the copy
Check the copy with "visudo"
If and only if visudo gives no error - overwrite the original file with modified copy

After sudoers file is updated there's no need to enter root password every time the purge is invoked. Hurray! 

Unfortunately this was not the end.
Application also had to display total and free memory and autostart on login.
Autostart is perfectly described on the internet? just google it. So I won't touch it here.
Total and free memory querying on the other hand is not described anywhere, so I'll also won't touch it here :) especially since I'm unhappy with the way I had to implement that part.
Ok, just kidding. I'll describe that next time I update this article.

