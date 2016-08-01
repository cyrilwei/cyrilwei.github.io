---
layout: post
title:  "The BigDecimal is not predictable too. A Trap?"
date:   2010/08/31 00:00:00 +0800
categories: java
published: false
---

Hi, have you used the `BigDecimal` class in java? And do you know that the `BigDecimal` is as unpredictable as `double`?

Recently, I failed in a test case which I thought would have never failed because I just returned the parameter in the method...ok, well, i converted it. So, what do you think the output is

{% highlight java %}
BigDecimal a = new BigDecimal(1.16);
BigDecimal b = new BigDecimal(100);
System.out.println(a.multiply(b).longValue());
{% endhighlight %}

It’s 115! Not the expected 116.

Oh, well, what’s wrong with it? It’s the precise, predictable and trustworthy `BigDecimal`. Don’t do that `double` things.

Well, `double`, yes, what’s the result of `double`?

{% highlight java %}
System.out.println((long)(1.16 * 100));
{% endhighlight %}

Bingo! 115 too. So, maybe the `BigDecimal` translates the exact `double` value, with the unpredictable behaviors of `double`.

{% highlight java %}
System.out.println(new BigDecimal(1.16));
{% endhighlight %}

The output is 1.1599999999999999200639422269887290894985198974609375. Yes, perhaps it’s the precise binary value of 1.16 in `double`.

To find the reason, I opened the Java SDK documents and, oops, there are some notes of the `BigDecimal(double val)` constructor.

> Notes:

> 1. The results of this constructor can be somewhat unpredictable. One might assume that writing new BigDecimal(0.1) in Java creates a BigDecimal which is exactly equal to 0.1 (an unscaled value of 1, with a scale of 1), but it is actually equal to 0.1000000000000000055511151231257827021181583404541015625. This is because 0.1 cannot be represented exactly as a double (or, for that matter, as a binary fraction of any finite length). Thus, the value that is being passed *in* to the constructor is not exactly equal to 0.1, appearances notwithstanding.
> 2. The String constructor, on the other hand, is perfectly predictable: writing new BigDecimal("0.1") creates a BigDecimal which is *exactly* equal to 0.1, as one would expect. Therefore, it is generally recommended that the <a href="http://download.oracle.com/javase/6/docs/api/java/math/BigDecimal.html#BigDecimal(java.lang.String)">String constructor</a> be used in preference to this one.
> 3. When a double must be used as a source for a BigDecimal, note that this constructor provides an exact conversion; it does not give the same result as converting the double to a String using the <a href="http://download.oracle.com/javase/6/docs/api/java/lang/Double.html#toString(double)">Double.toString(double)</a> method and then using the <a href="http://download.oracle.com/javase/6/docs/api/java/math/BigDecimal.html#BigDecimal(java.lang.String)">BigDecimal(String)</a> constructor. To get that result, use the static <a href="http://download.oracle.com/javase/6/docs/api/java/math/BigDecimal.html#valueOf(double)">valueOf(double)</a> method.

Ok, yes, that’s right. Well, then, why haven’t anybody told me this?

So the `BigDecimal` is trustworthy still, but don’t use the untrustworthy `double` any more. Use the `String` constructor, they recommended that.

{% highlight java %}
BigDecimal a = new BigDecimal("1.16");
BigDecimal b = new BigDecimal("100");
System.out.println(a.multiply(b).longValue());
System.out.println(a);
{% endhighlight %}

Very well, 116 and 1.16. But will you use `String` as number type in you applications? If you can’t, follow the 3rd note. The static method `valueOf(double)` will first convert the `double` to `String` and then use the `String` constructor.

Like this

{% highlight java %}
BigDecimal a = BigDecimal.valueOf(1.16);
BigDecimal b = BigDecimal.valueOf(100);
System.out.println(a.multiply(b).longValue());
System.out.println(a);
{% endhighlight %}

Output 116 and 1.16 too.

Ok, I chose `String` in my application because my data is money. I won’t use `double` type to store money data forever.

**Questions:**

1. Why haven’t mark a `'deprecated'` on the `BigDecimal(double val)` constructor?
2. What does the `double.toString(double d)` do to make the value reasonable?
