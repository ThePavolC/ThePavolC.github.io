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

# Code

I started with sample code that was generated when creating new project. Then I deleted everything I didn't need and this is the bare minimum I was left with.

{% highlight c %}
#include <pebble.h>

static Window *window;

static void window_load(Window *window) {}

static void window_unload(Window *window) {}

static void init(void) {
  window = window_create();
  window_set_window_handlers(window, (WindowHandlers) {
    .load = window_load,
    .unload = window_unload,
  });
  const bool animated = true;
  window_stack_push(window, animated);
}

static void deinit(void) {
  window_destroy(window);
}

int main(void) {
  init();
  app_event_loop();
  deinit();
}
{% endhighlight %}

We will use functions `window_load()` and `window_unload()` to create/destroy whole layout and later on to set up/destroy handlers.
This is the description about functions from [documentation][window-func]:

- `load`: called when the window is pushed to the screen when it's not loaded. This is a good moment to do the layout of the window.
- `unload`: called when the window is deinited, but could be used in the future to free resources bound to windows that are not on screen.

Text on screen is inside `TextLayer`, which is added then to root layer. So first I define `s_time_layer` variable in top of the file, then in `window_load()` we create and assign text layer to it. When creating text layer, you specify location and size. In this case I want this text layer to be at very top and to take the whole width of watch. To get width of screen I need to get bonds of `windows_layer` or root layer.

Then I use `text_layer_set_font()` to specify size and type of font I want to use. For now I stick with the [Pebble system fonts.][sys-fonts]
I also changed text alignment with `text_layer_set_text_alignment()`.

After I set text using `text_layer_set_text()`, all I need to do is to add new text layer to window's root layer, for that I use `layer_add_child()`.

{% highlight c %}
#include <pebble.h>

static Window *window;
static TextLayer *s_time_layer;

static void window_load(Window *window) {
    Layer *window_layer = window_get_root_layer(window);
    GRect bounds = layer_get_bounds(window_layer);
    s_time_layer = text_layer_create(GRect(0, 0, bounds.size.w, 42));

    text_layer_set_font(s_time_layer, fonts_get_system_font(FONT_KEY_BITHAM_42_BOLD));
    text_layer_set_text_alignment(s_time_layer, GTextAlignmentCenter);
    text_layer_set_text(s_time_layer, "12:34");

    layer_add_child(window_get_root_layer(window), text_layer_get_layer(s_time_layer));
}

{% endhighlight %}


[download-sdk]: http://developer.getpebble.com/sdk
[install]: http://developer.getpebble.com/sdk/install/linux/
[dev-connection]: http://developer.getpebble.com/guides/publishing-tools/developer-connection/
[emulator]: http://developer.getpebble.com/sdk/install/linux/#install-pebble-emulator-dependencies
[window-func]: http://developer.getpebble.com/docs/c/User_Interface/Window/#WindowHandlers
[sys-fonts]: http://developer.getpebble.com/guides/pebble-apps/display-and-animations/ux-fonts/#pebble-system-fonts
