---
layout: post
title: Flipping Window (OS X)
---
This is a quick note about window flipping effect under OS X.

I've found the code somwhere on the internet and I like how the effect looks (although, I don't like the code).

Basicly the process consists of these steps:

1. Render two window's views into layers
2. Create a temporary transparent window
3. Animate transforms of two rendered layers inside created window to emulate windows rotation
4. Visiblate/invisiblate base windows when needed

This is how it looks:

![]({{ site.baseurl }}/images/window-flip.gif)

Even though the source code is not pretty at all it is [worth looking at](https://github.com/golopupinsky/NSWindow-flip-demo). 