---
title: 2. Enable RPi UART on Arch Linux
layout: page
nav: true
---

# Enable RPi UART on Arch Linux

The Raspberry Pi 3 has the mini-UART mapped to the GPI header pins 14/15. By default, it is also configured as the serial console. The actual hardware UART on the BCM2837 SoC is assigned to handle Bluetooth with the BCM43438 Wifi/Bluetooth chip.

To enable the hardware UART on pins 14/15 we need to do the following:

```
$ sudo nano /boot/config.txt
```

Add the following lines 

```
dtoverlay=pi3-diable-bt
enable_uart=1
```

Note: the above will disable Bluetooth. Alternately the mini-UART can be mapped to the Bluetooth module. Change the `dtoverlay` line to `pi3-miniuart-bt`.

Next, we modify the cmdline.txt file to remove parameters involving serial port “ttyAMA0”.

```
$ sudo cp /boot/config.txt /boot/config_backup.txt
$ sudo nano /boot/config.txt
```

Lastly, we add the default user “alarm” to the same group as the serial port. In this case “uucp”

```
$ sudo usermod –a –G uucp alarm
```

Reboot and test the serial port with a simple loopback between pins 14 and 15. For this, I used Cutecom built for armv7h.
