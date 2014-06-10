---
layout: post
title: "Switch Frames in Selenium"
date: 2013-09-25 13:09
categories: selenium
---

I've noticed that I was getting inexplicable "ElementNotFound" errors in Selenium when
trying to swith frames after having previously switched a frame else where. It turns out
that you need to switch back to the "defaultContent" frame. 

For example, this **will not** sucessfully switch to "main\_frame\_2"
{% highlight java %}
driver.switchTo().frame("main_frame_1");
driver.switchTo().frame("sub_frame_of_frame_1");
driver.switchTo().frame("main_frame_2");
{% endhighlight %}

However, by using "defaultContent", this **does work**
{% highlight java %}
driver.switchTo().frame("main_frame_1");
driver.switchTo().frame("sub_frame_of_frame_1");
driver.switchTo().defaultContent();
driver.switchTo().frame("main_frame_2");
{% endhighlight %}
