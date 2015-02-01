---
layout: post
title: UIImageView activity indicator
---
This is a quick note on a really nice UIImageView feature I discovered recently.
It allows you to set up your custom activity indicator based on UIImageView. It makes use of ```animationImages``` property.

It works pretty easy. You just assign array of images to ```animationImages``` property and also set up animation duration and then call ```startAnimating```. And that's it.

To reduce the amount of images you should import into your project you can just apply simple transformations like rotations to one image. For example I took one image and rotated it 50 times producing more images. You can see the actual animation on the image below.

![]({{ site.baseurl }}/images/activity.gif)

Since I was not paying attention to keeping bounds constant I received nice zooming as a side effect.  

Check out my [github project](https://github.com/golopupinsky/ImageViewActivityIndicator) for actual code.


