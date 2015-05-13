---
layout: post
title:  "Rails error: Session contains objects  whose class definition isn’t available"
categories: jekyll update
---

I am writing a rails3 application where I am storing some custom
objects in the session. All goes well until I restart my server and
all the existing sessions start throwing this error:

<pre>
Session contains objects whose class definition isn't available.
Remember to require the classes for all objects kept in the session.
(Original exception: uninitialized constant Map [NameError])
</pre>


Specifically I have a geographical map class (Map) and I am storing
the map object in sessions. Rails cannot unmarshal the serialized
session representation into the map object until it knows what the Map
class looks like.

To overcome this problem, I added the following line of code in my
<strong>config/locales/application.rb</strong> file and it solved the
error for me:

{% highlight ruby %}
require "#{Rails.root}/app/models/map"
{% endhighlight %}


where my map class is defined in /app/models/map.rb file.

Though it is discouraged to store complex objects in sessions, I got
to know about this taboo much later in my development cycle. At that
point, this fix was easier for me than to rewrite major pieces of
code.

I hope this helps someone on the other side of planet who is
banging his head over this peculiar error.
