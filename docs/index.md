---
title: Home
---

<div> 
    <img src="{{ '/images/linuxcnc-wizard.gif' | absolute_url }}" alt="LinuxCNC">
    <img src="{{ '/images/RaspberryPi.jpg' | absolute_url }}" alt="RPi">
</div>

# LinuxCNC on RPi 

Home page for LinuxCNC on RPi

<div class="toc" markdown="1">
## Contents:

{% for topic in site.pages %}
{% if topic.nav == true %}- [{{ topic.title }}]({{ topic.url | absolute_url }}){% endif %}
{% endfor %}
</div>