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

Installing Pebble SDK was very simple for me and without any issues. [On download page][download-sdk] you can decide between new 3.0 SDK which is still in Beta version or 2.9 SDK, which is the one I picked. Pebble strongly recommends to rebuild and retest apps to all developers, but I doubt that mine simple watchface would need it at this point.

[Installation of SDK for Linux][install] is pretty much copy/past action. After that I wanted to check if everything works properly. First of all, you need to [enable developer connection][dev-connection], so you can try out your app on the watch. There is also [emulator][emulator], but I want real thing. In developer view on the phone app, there is server IP address, you will need it later.

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

# What I want to see in the watchface



[download-sdk]: http://developer.getpebble.com/sdk
[install]: http://developer.getpebble.com/sdk/install/linux/
[dev-connection]: http://developer.getpebble.com/guides/publishing-tools/developer-connection/
[emulator]: http://developer.getpebble.com/sdk/install/linux/#install-pebble-emulator-dependencies
