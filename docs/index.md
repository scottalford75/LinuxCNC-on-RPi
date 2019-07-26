---
title: Home
---

<div> 
    <img src="{{ '/images/linuxcnc-wizard.gif' | absolute_url }}" alt="LinuxCNC">
    <img src="{{ '/images/RaspberryPi.jpg' | absolute_url }}" alt="RPi">
</div>

# LinuxCNC on RPi 

<div class="toc" markdown="1">
## Contents:

{% for topic in site.pages %}
{% if topic.nav == true %}- [{{ topic.title }}]({{ topic.url | absolute_url }}){% endif %}
{% endfor %}
</div>

##

## Why LinuxCNC on Raspberry Pi?

I've played with LinuxCNC for quite some years, from right back when it was known as EMC2. When PC's with parallel ports were common it was very straighforward to get up an going with a simple stepper system.

Having a low cost and accessable hardware platform for LinuxCNC is important if we want to use LinuxCNC for 3D printing for example. Having a controller box the size of the printer itself makes no sense. A SoC based single board computer would be ideal for this application. The Beaglebone is proven in this area but is cost prohibitive. Is it possible for the humble RPi do do the job...

It has been perceived that the Raspberry Pi has not been a viable hardware for LinuxCNC due to several reasons:

- Realtime performance not great for `base-thread` step gernerators
- UI performace poor resulting in frustrating user experience

So what's changed...? 

### Official RPi Preempt-RT

Since 2018 there has now been an official RPI Preempt-RT kernel branch being maintained by Tiejun Chen. <https://github.com/raspberrypi/linux>

There is a great kernel building tuturial done be LeMaRiva. <https://lemariva.com/blog/2018/07/raspberry-pi-preempt-rt-patching-tutorial-for-kernel-4-14-y>

### QtPyVCP

The standard Axis UI for LinuxCNC placed a lot of load onto the RPi. There is now a new UI framework, QtPyVCP, that now has a VTK based G code backplot apparently is less resource heavy.

> I'm currently testing QtPyVCP and I have also developed a HalPlot widget for a 3D Printer UI

### spiPRU

I have developed **spiPRU** that turns a cheap 32bit 3D Printer control board into a PRU (programmable real-time unit) connected to the RPi via the SPI bus. This then gives hard real-time step generators for the RPi similar to the Beaglebone inbuilt RPU's.

For <$150 it's now possible to have a LinuxCNC system!