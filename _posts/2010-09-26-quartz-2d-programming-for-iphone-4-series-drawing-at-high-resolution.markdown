---
layout: post
title:  "Quartz 2D Programming for iPhone 4 Series: Drawing at High-Resolution"
date:   2010/09/26 00:00:00 +0800
categories: iOS
---

The iPhone 4 has a really high-resolution screen which’s got 960 by 640 pixels. So the apps now need to be prepared to run on different resolutions. Although the apple said that most of the work was done by the system framework, your application still needs to do some other works.~

The UIKit will render texts and standard views on the high-resolution correctly, and if the app provides the @2x files, the UIKit will display the high-resolution images too.

Instead of *pixels*, the Quartz and UIKit use ***points*** to measure distances. One point does not necessarily correspond to one pixel on the screen. In iOS 4, the UIScreen, UIView, UIImage and CALayer expose a scale factor that tells you the relationship between points and pixels for that particular object. This factor is set to 2.0 on iPhone 4, so the apps which think the screen as 480 by 320 points would work well.

But the coordinate system of iOS may make some troubles.

The origin in Mac OS X is located at the lower-left corner, but the coordinate system of iOS is different. Its origin is located at the upper-left. However, if you want to draw some bitmaps or PDFs on iOS, the origin will be located at the lower-left, just like the Mac OS X coordinate system. So, the only one different is the device coordinate system of iOS.

Actually, the default origin of iOS is lower-left too. If you use `CGContextGetCTM(context)` to get the default transformation matrix on the `drawRect:` method of your views, you will find that it’s not an identity matrix, which means the iOS has already done some translation on the coordinate system. Maybe this is for someone to port games from other platforms.

To make the custom drawing code more general, or just because the developers are more familiar with the lower-left corner, some of them would like to ‘reset’ the coordinate system by asking Quartz 2D to apply an inverse matrix to the current transformation. Like this.

{% highlight objc %}
CGAffineTransform t0 = CGContextGetCTM(context);
t0 = CGAffineTransformInvert(t0);
CGContextConcatCTM(context, t0);
{% endhighlight %}

Then the transformation will be reset, the context’s transformation matrix will be identity and the origins of everything are located at the lower-left now.

That sounds good. Everything is reset, including the scale transformation.

Before the iPhone 4, the default scale transformation is 1, which is harmless. But things changed. To avoid the custom drawing code in old apps making mistakes on iPhone 4, the iOS sets a scale transform whose value is 2 on the default transform matrix for the `drawRect:` method of your views. The code will reset it to 1, which means that all the custom drawing will be displayed in the lower-left quarter of the screen.

Like this.

![Asteroids running in quarter screen](/assets/images/2010/09/quarter.png)

To fix this bug, we can use the scale property of UIScreen which is 2 on iPhone 4 and 1 on other iPhones and maybe something else on some further devices.

Add 2 lines after the reset code.

{% highlight objc %}
CGAffineTransform t0 = CGContextGetCTM(context);
t0 = CGAffineTransformInvert(t0);
CGContextConcatCTM(context, t0);

CGFloat scale = [UIScreen mainScreen].scale;
CGContextScaleCTM(context, scale, scale);
{% endhighlight %}

This will reset the context to 480 by 320 points. Then all the custom drawing will be enlarged 2 times to fit the new screen’s resolution, but which means that the custom drawing cannot benefit too much from the high-resolution.

![Asteroids running in full screen](/assets/images/2010/09/full.png)

Actually, we want to use the high-resolution screen to get more beautiful pictures. So the custom drawing code should use the 960 by 640 screen sufficiently.

We can use the reset context directly, for it has a 960 by 640 points canvas. If you don’t reset the coordinate system, you can use the following code to set the scale of transformation to 1. Then we have 960 by 640 points too.

{% highlight objc %}
CGFloat inverseScale = (CGFloat)1.0 / [UIScreen mainScreen].scale;
CGContextScaleCTM(context, inverseScale, inverseScale);
{% endhighlight %}

The next step to do is to modify the custom drawing code to use the more points. We could only multiply the position of drawing codes by 2, but maybe some lines will be too thin on the high-resolution screen.

A better way is to provide 2 different codes for different resolutions. Just like the images, we can provide an ‘@2x’ drawing code. When finding it the iPhone 4, the app will draw with the optimized code for high-resolution screen.

The best solution depends on the needs of the application. We should test some approaches before we choose.

For further information please refer to the iOS Application Programming Guide. There is a chapter discussing supporting high-resolution screens.

\* The pictures was taken from *Asteroids* - a sample game from [*Beginning iPhone Games Development*](http://playcontrol.net/iphonegamebook/), which is a wonderful book for iPhone game developer.
