---
layout: post
title: Pebble Watchface
description:
categories: pebble watchface
published: False
---

Intro

1. how to install api and run basic check
2. define what we want to see
3. adding time labels
4. adding connection status
5. adding battery stats

Outro

# Install SDK and check connectivity

Installing Pebble SDK is very simple and it worked without any issues. [On download page][download-sdk] you can decide between new 3.0 SDK which is still in Beta version or 2.9 SDK, which is the one I picked. Pebble strongly recommends to rebuild and retest apps to all developers, but I doubt that mine simple watchface would need it at this point.

[Installation of SDK for Linux][install] is pretty much copy/past action. After that I wanted to check if everything works properly. First of all, you need to [enable developer connection][dev-connection], so you can try out your app on the watch. There is also [emulator][emulator] available. In developer view on the phone app, there is server IP address, you will need it later.

To test connection, you can create "Hello World" app and push it to your watch. Just go to your project folder and create new Pebble project. Then you can export your server IP address to env variables so you don't have to type it all the time. After that go to just created Pebble project and buil&install.

{% highlight bash %}
pavol@Jacob:~$ pebble new-project Test
Creating new project Test
pavol@Jacob:~$ cd Test/
pavol@Jacob:~$ export PEBBLE_PHONE=192.168.0.5
pavol@Jacob:~$ pebble build && pebble install
{% endhighlight %}

Sometimes I got this error and all I need to do is wake up phone.

{% highlight bash %}
[ERROR   ] [Errno 104] Connection reset by peer
{% endhighlight %}

# Design watchface

With every watchface I used before, there was always something missing. In my watchface I want to see as much time related information as possible.

The biggest priority is time and date with the day of the week. Another useful information is week of the year, which can be handy in work/business environment, and possibly the day of the year.

Last but not least, is the connection status and battery status. With battery status I want to know percentage even though it takes a bit more space. And with connection status, I'd like to see the last time when the connection status was changed. That could be handy in situation when you forget your phone somewhere and you want to see when was the last time you were connected.




[download-sdk]: http://developer.getpebble.com/sdk
[install]: http://developer.getpebble.com/sdk/install/linux/
[dev-connection]: http://developer.getpebble.com/guides/publishing-tools/developer-connection/
[emulator]: http://developer.getpebble.com/sdk/install/linux/#install-pebble-emulator-dependencies
