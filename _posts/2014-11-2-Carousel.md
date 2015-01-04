---
layout: post
title: Carousel
---
This is the basic example of iOS carousel view.
It is not as advanced as for example [iCarousel](https://github.com/nicklockwood/iCarousel) but it gives understanding of basics.

The idea behind carouselish-controlls is that elements move farther  in perspective during scrolling. 

There are two ways of implementing that behaviour:

1. Scale views as they go farther/closer
2. Manipulate layer's 3d-transform z coordinate instead

Although manipulating 3d-transform seems to be most natural way of implementing this kind of effect it is not strictly the best one. 
Scaling allows doing fancy effects like size shivering, disproportional size change, and so on. On the other hand scaling requires you managing z buffer (i.e. arranging views on top of each other) on your own, while 3d-transform approach does that automagically.

We'll go with manipulation z coordinate as this seems to be the best way of understanding what exactly happens.

First of all, create view class and add a ```CGPoint panDistance``` variable to it. Also add a ```UIPanGestureRecognizer```. The action of gesture recognizer should look like this:


{% highlight objc %}
-(void)panned:(UIPanGestureRecognizer*)pan
{
    static CGPoint panStart;
    
    if(pan.state == UIGestureRecognizerStateBegan)
    {
        panStart = panDistance;
    }
    
    if(pan.state == UIGestureRecognizerStateChanged)
    {
        CGPoint t = [pan translationInView:self];
        panDistance = CGPointMake(panStart.x + t.x, panStart.y + t.y);
        [self layoutSubviews];
    }
}
{% endhighlight %}


This method will store current pan distance and also call ```layoutSubviews``` when needed. Let's look at ```layoutSubviews``` implementation.

{% highlight objc %}
-(void)layoutSubviews
{
    [super layoutSubviews];
    for ( int i=0; i<subviews.count; i++) {
        UIView* view = subviews[i];
        view.layer.transform = [self transformForViewAtIndex:i];;
    }
}
{% endhighlight %}

Not too much here, just setting a layer's transform for every view in subviews array. I do not provide code for initializing subviews array since I assume you can write it on your own (or just get full project [here](https://github.com/golopupinsky/Carousel)). But I discourage you from iterating over view's sibviews property here, because their ordering may change and some of them may not be the moving subviews of carousel. You should manage subviews array on your own.

So, ```layoutSubviews``` method calls ```transformForViewAtIndex:``` providing index of subview.

{% highlight objc %}
-(CATransform3D)transformForViewAtIndex:(NSUInteger)index
{
    CATransform3D transform = CATransform3DIdentity;
    transform.m34 = -1 / 1000;
    CGRect screenBounds = [UIScreen mainScreen].bounds;
    float yTranslation = CGRectGetHeight(screenBounds)/2 - SUBVIEW_SIZE/2;
    
    transform = CATransform3DTranslate(transform,
                                       [self xTranslation:index],
                                       yTranslation,
                                       [self zTranslation:index]
                                       );
    return transform;
}
{% endhighlight %}

First, ```transformForViewAtIndex:``` sets ```m34``` member of transform structure. This variable is responsible for perspective transforms - it controlls how fast element shrinks depending on distance to it. I am not going to describe this here, you can read more for example [here](http://milen.me/writings/core-animation-3d-model/).
Second, we can see that ```transformForViewAtIndex:``` depends on two other methods: ```xTranslation:``` and ```zTranslation:```. So far we have not seen anything that could be responsible for calculating subviews coordinates. All the magic should be inside those methods. And it is indeed there.


{% highlight objc %}
-(float)xTranslation:(NSUInteger)index
{
    float screenW = CGRectGetWidth([UIScreen mainScreen].bounds);
    float screenCenter = screenW / 2;
    float initialPhase = 2 * M_PI / [self count] * index;
    float panPhase = 2 * M_PI * panDistance.x / screenW * 2;
    
    return screenCenter + sinf(initialPhase + panPhase) * screenW/3 - SUBVIEW_SIZE/2;
}

-(float)zTranslation:(NSUInteger)index
{
    float screenW = CGRectGetWidth([UIScreen mainScreen].bounds);
    float initialPhase = 2 * M_PI / [self count] * index;
    float panPhase = 2 * M_PI * panDistance.x / screenW * 2;
    
    return cos(initialPhase + panPhase) * screenW/10;
}
{% endhighlight %}

These methods are very similar. They actually contain copy-pasted code. Ouch! This should be rewritten! Yes, of course. But for the sake of simplicity of this tutorial I decided to leave things as they are. 

Let's look at the common part of methods - variables.

* screenW - screen width. Note: since iOS 8 this will be orientation dependant which is prety cool.
* initialPhase - calculated based on index of subview, total subviews count and the fact that full circle is 2π radians. This one defines the initial position of subview.
* panPhase - depends on panDistance.x which we set in the beginning. Defines views positions depending on current pan distance.

Now that we know how variables are computed we can look at the differences of two methods.

First of all, x coordinate depends on sine and z depends on cosine. This is easy to understand by looking at extreme values.

<table class="black-table-centered">
  <tr>
    <td>1</td>    <td>2</td>                                        
  </tr>
    <tr>
    <td>3333</td>    <td>42112313123</td>                                        
  </tr>
    <tr>
    <td>qwe</td>    <td>rty</td>                                        
  </tr>
    <tr>
    <td>qwe</td>    <td>rty</td>                                        
  </tr>
</table>


<center>
<div>
<p>

| sin            | cos          |
|----------------|--------------|
| sin(0) = 0     | cos(0) = 1   |
| sin(π/2) = 1   | cos(π/2) = 0 |
| sin(π) = 0     | cos(π) = -1  |
| sin(3π/2) = -1 | cos(3π/2) = 0|

</p>
</div>
</center>

Since we want our views to go farther from us as we scroll we're using cosine because it decreases on interval [0;π]. On the other hand we want x coordinate to increase with scrolling only on half of that interval, i.e. [0;π/2] and then decrease on [π/2;3π/2] which is exactly what sine does. 

We still have a little bit more complex x coordinate computations because of the fact that we have to start from the center of the screen (we add screenCenter) and also because rotating view initial coordinate is always at the top left corner and not in center (subtracting SUBVIEW_SIZE/2 to account for that).

And that's it. This is all the basics you need to implement your own carousel view.  

The last thing we haven't discussed is movement decceleration after gesture finishes. Again, for the sake of simplicity we'll use timer for that (though that's probably the worst possible solution).
Add following code to the end of ```panned``` mathod.

{% highlight objc %}
    if(pan.state == UIGestureRecognizerStateEnded || 
    	pan.state == UIGestureRecognizerStateCancelled)
    {
        CGPoint v = [pan velocityInView:self];
        
        fadeTimer = [NSTimer 	timerWithTimeInterval:0.05 
        						target:self
        						selector:@selector(panFade) 
        						userInfo:[NSValue valueWithCGPoint:v] 
        						repeats:YES];
        						
        [[NSRunLoop mainRunLoop]addTimer:fadeTimer 
        			forMode:NSRunLoopCommonModes];
    }
{% endhighlight %}

Also, add ```[self stopFading]``` call to the beginning of ```if(pan.state == UIGestureRecognizerStateBegan)``` clause of ```panned``` mathod.

Lastly, add two methods:
 
* panFade - for changing the ```panDistance``` for some time after gesture ends
* stopFading - for invalidationg timer and cleaning up

Try implementing those on your own as an exercise.

Feel free to grab full project [source](https://github.com/golopupinsky/Carousel) from github. 

*Also, make sure to check out [tweaks-branch](https://github.com/golopupinsky/Carousel/tree/tweaks-branch) of the project. It's more advanced and cleaner than master branch. Note: it uses cocoapods, so only open project with '.xcworkspace' file.*


Remember, current implementation is very basic. It lacks many important things and serves only educational purposes. Do not use it in production.