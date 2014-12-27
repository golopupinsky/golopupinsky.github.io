---
layout: post
title: Circular Slider
---
Here I'd like to talk about circular sliders in general and gradient circular sliders in particular.

Very nice circular slider example is located [here](https://github.com/ariok/TB_CircularSlider), I based mine on it.

I do not like couple things about that slider though. One of those is that it does not allow drawing circular gradient.

Currently there is no 'out-of-the-box' way of drawing circular gradients on iOS, so we'll have to do that manually. The basic idea here is to draw many small rectangles close to each other,tinting each one with its own color. This approach allows almost any shapes and colors variations. 

There's a nice [SO discussion](http://stackoverflow.com/questions/11783114/draw-outer-half-circle-with-gradient-using-core-graphics-in-ios) on this topic. One minor improvement there is that rectangles are replaced with trapezoids. This way shapes won't overlap with corners. 

Long story short, here's slightly refined SO code.

```
		//1
        int subdiv=512;
        float interiorPerim = M_PI*radius;
        float exteriorPerim = M_PI*(radius+WIDTH);
        float smallBase= interiorPerim/subdiv;
        float largeBase= exteriorPerim/subdiv;
        
        //2.1
        UIBezierPath *cell = [UIBezierPath bezierPath];
        [cell moveToPoint:CGPointMake(-smallBase/2,radius-WIDTH/2)];
        [cell addLineToPoint:CGPointMake(smallBase/2,radius-WIDTH/2)];
        [cell addLineToPoint:CGPointMake(largeBase/2,radius+WIDTH)];
        [cell addLineToPoint:CGPointMake(-largeBase/2,radius+WIDTH)];
        [cell closePath];

        //2.2
        float incr = 2 * M_PI / subdiv; //increment angle
        CGContextTranslateCTM(	ctx, 
        						CGRectGetWidth(self.bounds)/2, 
        						CGRectGetHeight(self.bounds)/2);
        CGContextRotateCTM(ctx, M_PI/2);
        CGContextRotateCTM(ctx,-incr/2);

		//3        
        for (int i=0;i<subdiv;i++) {
            UIColor *c = [UIColor colorWithHue:(float)i/subdiv saturation:1 brightness:1 alpha:1];
            [c set];
            [cell fill];
            [cell stroke];
            CGContextRotateCTM(ctx, -incr);
        }


```

This is an incomplete excerpt from slider drawRect method.  [Full project](https://github.com/golopupinsky/CircularSlider).

Let's see what is going on here.

1. Trapezoid initialization

	Here we initialize the amount of trapezoids to use (subdiv variable) and small and large trapezoid's bases (we need to calculate inner and outer perimeters of slider ring before we can find bases).

2. Bezier path building
	
	Forming bezier path of a trapezoid.

3. Positioning path

	Settin up initial trapezoid position and rotation.
	
4. Stroking trapezoids
	
	Drawing trapezoid path rotating it as necessary.


By varying the subdiv variable we can achieve various results.
	
![]({{ site.baseurl }}/images/rgCro.png)

(image borrowed from aforementioned SO thread)

The [slider](https://github.com/golopupinsky/CircularSlider) we were working at looks like this

![]({{ site.baseurl }}/images/circular-slider.png)
