---
layout: post
title: Pebble Watchface
description: Simple tutorial on how to make Pebble watchface.
categories: pebble watchface
published: True
---

Since I've had Pebble, I always wondered what is the API like and how difficult it is to do your own app or watchface. Well it turns out that it's quiet easy especially when you have good documentation and bit of a time.

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

I also modified `appinfo.json` file where I changed `watchaface` value to `true`. You could also change the name of the watchface, add your company name, and so on.

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

This is what I ended up with.

![Pebble Watchface 1](/assets/pebble-watchface/screenshot1.png)

So next I added all other text layers for date info, battery and bluetooth status. Regarding bluetooth and battery status, at this point I am adding only text layers but later on I'll add icons.

And this is what my watchface looks like with all text layers.

![Pebble Watchface 2](/assets/pebble-watchface/screenshot2.png)

Next I'd like to add some icons for bluetooth and battery status. I have bluetooth icon of size 24x24 pixels, which was big enough to see all details on watch but still small. I copied icon to `resources` folder and added it to `appinfo.json` as new resource.

{% highlight json %}
"resources": {
    "media": [
        {
            "type": "png",
            "name": "BLUETOOTH_ICON_ACTIVE",
            "file": "bluetooth-active.png"
        }
    ]
  } 
{% endhighlight %}

To add icon to screen you need to use [`GBitmap`][gbitmap] and [`BitmapLayer`][bitmaplayer]. It's fairly simple, first you create new `GBitmap` with your icon as resource. To access your resource you append `RESOURCE_ID_` to the icon's name from `appinfo.json` file. In my case it is `RESOURCE_ID_BLUETOOTH_ICON_ACTIVE`. Then you create bitmap layer and you set bitmap for that layer, which is GBitmap.
Finally, add new bitmap layer to window layer.

{% highlight c %}
...
static GBitmap *s_bluetooth_bitmap_active;
static BitmapLayer *s_bluetooth_layer;
...

static void window_load(Window *window) {
...
s_bluetooth_bitmap_active = gbitmap_create_with_resource(RESOURCE_ID_BLUETOOTH_ICON_ACTIVE);
s_bluetooth_layer = bitmap_layer_create(GRect(0, 140, 22, 22));
bitmap_layer_set_bitmap(s_bluetooth_layer, s_bluetooth_bitmap_active);

layer_add_child(window_get_root_layer(window), bitmap_layer_get_layer(s_bluetooth_layer));
...
{% endhighlight %}

And here we can see the new icon.

![Pebble Watchface 3](/assets/pebble-watchface/screenshot3.png)

In my watchface I use two icons for bluetooth for connected and disconnected. For battery I use five icons, where one is used while charging and rest is used to show battery level. You can see the code for my watchface at [my GitHub repo][pebble-watchface-git].

Now we can finally write code to show the time, I mean it is a watch after all. Pebble uses services for time, bluetooth, battery stats and in code we subscribe functions to those services.

First, lets subscribe to `tick_timer_service` with  `MINUTE_UNIT` and `tick_handler` function. This means that every minute that function will be executed.

{% highlight c %}
static void tick_handler(struct tm *tick_time, TimeUnits units_changed) {
    update_time();
    update_date_day_week();
}

static void init(void) {
  ...
  tick_timer_service_subscribe(MINUTE_UNIT, tick_handler);
}
{% endhighlight %}

Then in functions `update_time` and `update_date_day_week` I will overwrite text layers with time/date information.

First I will obtain time in seconds since the epoch, then I convert that time to `struct tm` which contains the time adjusted for the local time zone. Then I can use `strftime` which will format time value and place the result in buffer.

[You can find all format specifiers for `strftime` here.][strftime]

{% highlight c %}
static void update_time() {
    time_t temp = time(NULL);
    struct tm *tick_time = localtime(&temp);

    static char time_buffer[] = "00:00";

    if (clock_is_24h_style() == true) {
        strftime(time_buffer, sizeof("00:00"), "%H:%M", tick_time);
    } else {
        strftime(time_buffer, sizeof("00:00"), "%I:%M", tick_time);
    }

    text_layer_set_text(s_time_layer, time_buffer);
}

static void update_date_day_week() {
    time_t temp = time(NULL);
    struct tm *tick_time = localtime(&temp);

    static char date_buffer[] = "ABC, 00 000 0000";
    strftime(date_buffer, sizeof("ABC, 00 000 0000"), "%a, %d %b %Y", tick_time);
    text_layer_set_text(s_date_layer, date_buffer);

    static char day_week_buffer[] = "day: 000 week: 00";
    strftime(day_week_buffer, sizeof("day: 000 week: 00"), "day: %j week: %W", tick_time);
    text_layer_set_text(s_day_week_layer, day_week_buffer);
}
{% endhighlight %}

Time and date info should be correct by now. So let's focus on bluetooth status. I assume you have two icons for connected and disconnected prepared and available as resources.

What we need to do is subscribe to bluetooth service and check if we are connected or not, based on that we show different icon. In my function I also set the text layer next to the bluetooth icon to the time when the bluetooth status change happened. And to make sure I notice the change, I add the `vibes_double_pulse` function call, which will vibrate the watch.

{% highlight c %}
static void bluetooth_connection_handler(bool connected) {
    vibes_double_pulse();

    if (connected) {
        bitmap_layer_set_bitmap(s_bluetooth_layer, s_bluetooth_bitmap_active);
    } else {
        bitmap_layer_set_bitmap(s_bluetooth_layer, s_bluetooth_bitmap_inactive);
    }

    time_t temp = time(NULL);
    struct tm *tick_time = localtime(&temp);
    static char time_buffer[] = "00:00";

    if (clock_is_24h_style() == true) {
        strftime(time_buffer, sizeof("00:00"), "%H:%M", tick_time);
    } else {
        strftime(time_buffer, sizeof("00:00"), "%I:%M", tick_time);
    }

    text_layer_set_text(s_bluetooth_time_layer, time_buffer);
}


static void init(void) {
  ...
  bluetooth_connection_service_subscribe(bluetooth_connection_handler);
  bluetooth_connection_handler(bluetooth_connection_service_peek());
}
{% endhighlight %}

The last thing I added was battery status. I have 5 icons prepared for different battery levels and also to show that battery is being charged. Now I need to do similar thing as with bluetooth. I need to subscribe to `battery_state_service` with my `battery_state_handler` function.

In `battery_state_handler` function I will check what's the current state and display correct icon. I also add current percentage to the text layer right next to the icon.

{% highlight c %}
static void battery_state_handler(BatteryChargeState charge_state) {
    static char s_battery_buffer[4];

    if (charge_state.is_charging)
    {
        bitmap_layer_set_bitmap(s_battery_layer, s_battery_charging_bitmap);
    }
    else if (charge_state.charge_percent == 100) {
        bitmap_layer_set_bitmap(s_battery_layer, s_battery_100_bitmap);
    }
    else if (charge_state.charge_percent >= 80) {
        bitmap_layer_set_bitmap(s_battery_layer, s_battery_080_bitmap);
    }
    else if (charge_state.charge_percent >= 60) {
        bitmap_layer_set_bitmap(s_battery_layer, s_battery_060_bitmap);
    }
    else {
        bitmap_layer_set_bitmap(s_battery_layer, s_battery_040_bitmap);
    }

    snprintf(s_battery_buffer, sizeof(s_battery_buffer), "%d%%", charge_state.charge_percent);
    text_layer_set_text(s_battery_status_layer ,s_battery_buffer);
}

static void init(void) {
  ...
  battery_state_service_subscribe(battery_state_handler);
  battery_state_handler(battery_state_service_peek());
}
{% endhighlight %}

Finally we should end up with something like this.

![Pebble Watchface 4](/assets/pebble-watchface/screenshot4.png)

You can try to turn off your bluetooth and see if the icon will change, or you can try to charge your watch and battery icon should change.

There is one last piece of code that we should write, and that is to free resources. So in function `window_unload` we destroy all layers and unsubscribe from services.

{% highlight c %}
static void window_unload(Window *window) {
    text_layer_destroy(s_time_layer);
    text_layer_destroy(s_date_layer);
    text_layer_destroy(s_day_week_layer);
    gbitmap_destroy(s_bluetooth_bitmap_active);
    gbitmap_destroy(s_bluetooth_bitmap_inactive);
    bitmap_layer_destroy(s_bluetooth_layer);
    text_layer_destroy(s_bluetooth_time_layer);
    tick_timer_service_unsubscribe();
    bluetooth_connection_service_unsubscribe();
    battery_state_service_unsubscribe();
}
{% endhighlight %}

# Conclusion

Since I have created this watchface, it's been only one I've been using. I have created it and it's pretty much what I need.

There are few things I will change though. I am thinking of adding:

- weather forecast
- showing timezone
- using different font
- ...

I am also not quiet sure about the structure of my code. I think I am doing some calls in the wrong places. So there is a chance that I will rewrite the whole thing.

But if you are interested in the code anyway, you can find it all in my [PebbleWatchface repository.][pebble-watchface-git]

[download-sdk]: http://developer.getpebble.com/sdk
[install]: http://developer.getpebble.com/sdk/install/linux/
[dev-connection]: http://developer.getpebble.com/guides/publishing-tools/developer-connection/
[emulator]: http://developer.getpebble.com/sdk/install/linux/#install-pebble-emulator-dependencies
[window-func]: http://developer.getpebble.com/docs/c/User_Interface/Window/#WindowHandlers
[sys-fonts]: http://developer.getpebble.com/guides/pebble-apps/display-and-animations/ux-fonts/#pebble-system-fonts
[gbitmap]: http://developer.getpebble.com/docs/c/Graphics/Graphics_Types/#GBitmap
[bitmaplayer]: http://developer.getpebble.com/docs/c/User_Interface/Layers/BitmapLayer/
[pebble-watchface-git]: https://github.com/ThePavolC/PebbleWatchface
[strftime]: http://www.cplusplus.com/reference/ctime/strftime/
