---
title: "Controlling keyboard leds using Xlib"
categories: blog
excerpt: "Controlling keyboard leds using Xlib"
tags: [programming,xlib,keyboard,cpp]
comments: true
description: How to control keyboard leds using Xlib in C++  
---

I was in the computer lab, learning to code a simple concurrent echo server and I asked my professor if I can use C++,he nodded yes but he also said that C is elegant and much easier for stuffs like port programming. C is usually good for low-level stuff but C++ isn't bad either. Later simple discussion between C and C++ lead to a challenge which my professor gave me. He recollected that he wrote a C program to control keyboard leds in his college days and challenged (more of a friendly challenge) to accomplish the same. While I managed to write one in C++ but it is more of a wrapper of API calls to Xlib which is a C library :sweat_smile:. My first thought was to use setleds or tweaking proc files but that would be cheating as calls would be wrapped inside system() parameters. So after searching I came across use of Xlib to accomplish the same, [here](https://logixoul.wordpress.com/2008/08/14/how-to-set-leds-in-x-even-on-ubuntu/). I modified my code upon the work of [logixoul](https://logixoul.wordpress.com/2008/08/14/how-to-set-leds-in-x-even-on-ubuntu/) and found something difficult to comprehend (also found the documentation of XKBlib to be of little help, or maybe I was wrong somewhere). So I tried hacking around and got my result.
```
SCROLL_LOCK = 1
CAPSLOCK = 2
NUMLOCK = 16
```
are some predefined constants to be used by XkbLockModifiers. Most surprising was passing the bit manipulated result of these constants (can get how it changes, but it does play some part as changing them broke the result ).
I added some C++ 14 goodness, which was not needed and was more of an over-enginerring.

- auto instead of ```Display *dpy ```
- Use of thread for sleeping
- autofied parameter which was frankly not needed
- Trailing return type because why not :sunglasses:

<hr>

Anyways here goes the code, might not the best one but works. Toggles caps lock and num lock within an interval of 1s, infinetly.
Compile with following  Flag parameters **-std=c++14 -lX11 -lpthread**

{% highlight cpp %}
// Compile it  following flags -std=c++14 -lX11 -lpthread  
// Based upon https://logixoul.wordpress.com/2008/08/14/how-to-set-leds-in-x-even-on-ubuntu/
# include <X11/Xlib.h>
# include <X11/XKBlib.h>
# include <thread>
# include<iostream>

using namespace std;

const int  SCROLL_LOCK = 1,CAPSLOCK = 2,NUMLOCK = 16;

// Also tried to use lambdas but got into difficulties, so commented them out
//auto turnon = [&](){ return XkbLockModifiers(display,XkbUseCoreKbd,CAPSLOCK |  NUMLOCK,led&(CAPSLOCK |  NUMLOCK));};
//auto turnoff = [&](){ XkbLockModifiers(display,XkbUseCoreKbd,CAPSLOCK |  NUMLOCK,led&(CAPSLOCK |  NUMLOCK));};

void set_led(auto led,bool turn_on)
{

    auto display = XOpenDisplay(nullptr);

    if(turn_on)
    {
        XkbLockModifiers(display, XkbUseCoreKbd,  CAPSLOCK | NUMLOCK,led & (CAPSLOCK |NUMLOCK));
    }
    else
    {
        XkbLockModifiers(display, XkbUseCoreKbd,  CAPSLOCK | NUMLOCK,led & (CAPSLOCK &NUMLOCK));
    }
    XFlush(display);

    XCloseDisplay(display);
}
auto main() -> int
{
    while(true)
    {
        set_led(CAPSLOCK,true);
        this_thread::sleep_for(1s);
        set_led(CAPSLOCK,false);
        set_led(NUMLOCK,true);
        this_thread::sleep_for(1s);
        set_led(NUMLOCK,false);
        this_thread::sleep_for(1s);
        set_led(CAPSLOCK,true);
        set_led(NUMLOCK,false);
        set_led(CAPSLOCK,false);

    }
    return 0;
}

{% endhighlight cpp %}
