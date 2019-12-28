---
title: 1. LinuxCNC install
nav: true
---

# LinuxCNC install on Arch Linux - Raspberry Pi

LinuxCNC development and installation instructions are Debian based. Therefore building LinuxCNC on Raspbian Stretch is reasonably straightforward as we are working in a Debian based distribution. 

Why Arch Linux then you may ask? Well, Raspbian Stretch worked great for an out of the box LinuxCNC build, but once I wanted to use QtPyVCP to develop a custom 3D printer UI I soon found that the stable release model would not work correctly due to older OpenGL libraries. 

Therefore, the search was on to find a distribution that would give this functionality on the Raspberry Pi. All the Debian based distros appeared to have the same problem. Arch Linux looked to be an option and pleasingly a fully functional Raspberry Pi 3 with LinuxCNC and QtPyVCP was the result.

## To install Arch Linux

Arch Linux is a lightweight Linux distribution with rolling updates. Unlike Raspbian there is no prepared SD card image available so the initial installation is a little more involved. There is good installation instruction of the Arch Linux Arm site:

<https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3>
<https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4>

> *TODO* – script use <http://eyesfreelinux.ninja/posts/raspberry-pi-arch-and-the-fix.html> to create an image. This script also needs bsdtar, kpartx, and parted.

Once the base system is installed we need to get everything setup ready to install LinuxCNC, Qt5 and PyQT5.

  
### STEP 1: Get onboard Wifi up and running

Later we will setup Network Manager but initially we will configure wpa_supplicant to get the Wifi interface wlan0 up and running.

```
$ cd /etc/wpa_supplicant/  
$ nano homenetwork`
```

```
network={  
    ssid=“your ssid”  
    psk=“your pass code”    
}  
```

Start up the Wifi by specifying the interface, wlan0, and the configuration file we just created above.

```
$ wpa_supplicant –iwlan0 –c /etc/wpa_supplicant/homenetwork &
```

Manually start the DHCP daemon on the wireless interface.

```
$ dhcpcd wlan0
```


### STEP 2: Update the system

Now that the network is up, we can update the system.

```
$ pacman-key –-init
$ pacman-key --populate archlinuxarm
$ pacman -Syu
```


### STEP 3: Set user “alarm” as super user

Firstly, we need to install the sudo package.

```
$ pacman -S sudo
```

The default user is “alarm” so we need to setup super user privileges.

```
$ cd /etc/sudoers.d/
$ nano myOverrides
```

```
alarm ALL=NOPASSWD: ALL
```

For the remainder of the installation we will switch over to the default user “alarm”

```
$ su alarm
```


### STEP 4: Install X11 and desktop environment

Install X11 and the Xfce desktop environment.

```
$ sudo pacman –S xorg-server xf86-video-fbdev xorg-xrefresh
$ sudo pacman –S xfce4
```

If you want to keep a lightweight environment, you may want to skip the following and only install the utility software that you will need later.

```
$ sudo pacman –S xfce4-goodies
```

If not using a session manager, install xorg-xinit. 

```
~/.xinitrc
```

```
exec startxfce4
```


### STEP 5: Install session manager

Install a session manager. We will use SDDM (Simple Desktop Display Manager).

```
$ sudo pacman –S sddm
$ sudo sh –c “sddm --example-config > /etc/sddm.conf”
$ sudo systemctl enable sddm
$ sudo systemctl start sddm
```

Alternatively we can use lightdm.

```
$ sudo pacman -S lightdm lightdm-gtk-greater
$ sudo systemctl enable lightdm
$ sudo systemctl start lightdm
```
You should now be ready to login to the desktop environment. User “alarm” with password “alarm”.


### STEP 6: Install Network Manager

Open a terminal, install Network Manager, and start it.

```
$ sudo pacman –S networkmanager network-manager-applet
$ sudo systemctl enable NetworkManager
$ sudo systemctl start NetworkManager
```

In the top right hand of the screen, you will now have access to wired and wireless network configuration.


### STEP 7: Install development tools

Install some tools to enable compiling etc

```
$ sudo pacman –S make gcc git-core automake autoconf pkg-config
$ sudo pacman –S libtool binutils patch
```


### STEP 8: Install a desktop theme

If you like, you can install a desktop theme. For a nice dark theme, try ARC

```
$ sudo pacman –S arc-gtk-theme
```


### STEP 9: Enable auto mounting of USB media

It is handy to be able to auto mount USB media rather than having to ```sudo mount``` all the time. With Thunar file manager installed we can add the ```thunar-volman``` plugin. However, this did not appear to be available for armv7h so had to be built from source.

> `thunar-volman` now appears to be in the repos...

Several other packages are also needed as well.

```
$ sudo pacman –S autofs gvfs udisks2 ntfs-3g
```


### STEP 10: Install Python 2

We only want to install Python2 at this stage. With both Python 2 and 3 installed in the system, I had issues getting PyQt5 to make the correct plugin module. 

> As a note, avoid using Pip to install Python packages! Pacman and Pip don't seem to play well together.

```
$ sudo pacman –S python2
```


## To install TigerVNC

Accessing the Raspberry Pi from another computer is an easy way to use the Pi. The Virtual Network Computing (VNC) protocol will be used with the TigerVNC package.

```
$ sudo pacman –S tigervnc
```

### Create environment, config, and password files

The first time vncserver is run, it creates its initial environment, config, and user password file. These will be stored in `~/.vnc`, which will be created if not present.

```
$ vncserver
```

```
You will require a password to access your desktops.

Password:
Verify:

New 'alarmpi:1 (facade)' desktop is alarmpi:1

Creating default startup script /home/alarm/.vnc/xstartup
Starting applications specified in /home/alarm/.vnc/xstartup
Log file is /home/alarm/.vnc/alarmpi:1.log
```

We will use “System mode” to start the VNC server as required on the headless system. 

Create `/etc/systemd/system/vncserver@:1.service`, where :1 is the 5900 port increment (5900 + 1) to which the VNC server will be listening for connections (e.g vncserver@:5.service means the server will be listening to port 5905). Edit the service unit by defining the user (User=) to run the server, and the desired Vncserver options.

```
$ sudo nano /etc/systemd/system/vncserver@:1.service

[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=simple
User=alarm
PAMName=login
PIDFile=/home/%u/.vnc/%H%i.pid
ExecStartPre=/bin/sh -c '/usr/bin/vncserver -kill %i > /dev/null 2>&1 || :'
ExecStartPre=/usr/bin/dbus-launch
ExecStart=/usr/bin/vncserver %i -geometry 1440x900 -alwaysshared -fg
ExecStop=/usr/bin/vncserver -kill %i

[Install]
WantedBy=multi-user.target
```

Start and enable the service to rat at boot time / shutdown.

```
$ sudo systemctl enable vncserver@:1.service
$ sudo systemctl start vncserver@:1.service
```

We also need to setup TigerVNC to start our desktop environment when we connect remotely.  Modify  `/home/alarm/.vnc/xstartup` to read:

```
#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec dbus-launch startxfce4
```

The Raspberry Pi can now be accessed via a VNC client using alarmpi:1 as the VNC server to connect to.


## To install LinuxCNC

In the Arch Linux AUR (Arch User Repository) there is a package LinuxCNC-sim. This will build a “uspace” version of LinuxCNC into a package for installation. The dependencies and steps in the PKGBUILD were used as a starting point to build a run-in-place installation of LinuxCNC. 


### STEP 1: Install dependencies

Unfortunately there are not armv7h versions of all required dependencies available so we will need to build and install these first.

Arch Linux dependencies needed by LinuxCNC:

|Package| Repo Install |  PKGBUILD  |
|:---|:---:|:---:|
|intltool | Yes ||
|bc | Yes ||
|bwidget |  | Build|
|tcllib |  | Build|
|tclx |  | Build|
|tcl | Yes ||
|tk                |Yes ||
|xorg-server      |Yes ||
|python2-imaging | Yes ||
|tkimg |  | Build|
|python2-gtkglext |
|tclx |  | Build|
|boost | Yes ||
|boost-libs | Yes ||
|libtirpc | Yes ||
|procps-ng | Yes ||
|psmisc	| Yes ||

The fakeroot package is needed to build Arch Linux packages from a PKGBUILD.

```
$ sudo pacman –S fakeroot
```

New versions of LinuxCNC no longer include yapps in the source. Therefore, we need to install it in the system. However, Arch Linux does not seem to have a repo for this. I have included a Yapps source in the [repository](https://github.com/scottalford75/LinuxCNC-on-RPi/tree/master/ArchLinux)

```
/usr/bin
```

```
yapps
```

In the above python file make sure that it is pointing at python 2. The first line should read

```
#!/usr/bin/python2
```

The following yapps directory in `site-packages` must include the files listed.

```
/usr/lib/python2.7/site-packages/yapps
```

```
__init__
__init__.pyc
grammar
grammar.pyc
parsetree
parsetree.pyc
runtime
runtime.pyc
```


## STEP 2: Get LinuxCNC source files

You need to have git installed and configured. Then

```
$ git clone https://github.com/LinuxCNC/linuxcnc.git linuxcnc-dev
$ cd linuxcnc-dev
```


## STEP 3: Fix Python 2 naming

Arch Linux explicitly names both Python 2 and Python 3, whereas Debian systems use the Python and Python 3 naming convention. The following command will find the required files and change python to python2 within them.

```
$ find . -iname fixpaths.py -o -iname checkglade -o -iname update_ini|xargs perl -p -i -e "s/python/python2/"
```

> **Note:** run this command only once other wise you will end up with python22, python222 etc, and the compile will fail.


## STEP 4: Apply libtirpc patch

The makefile needs a patch applied so that the make will succeed on Arch Linux. Copy `libtirpc.patch` into the LinuxCNC `/src` directory and execute the following command.

```
$ patch -Np2 -i libtirpc.patch
```


## STEP 5: Run the autogen scripts

```
$ ./autogen.sh
```


## STEP 6: Run the configuration script

Run the configuration script with the following parameters. Note, if you need `libmodbus` this dependency can be installed. For armv7h this will need to be built from source.

```
$ ./configure --with-realtime=uspace --without-libmodbus --with-python=/usr/bin/python2.7 --enable-non-distributable=yes
```

The script should complete without errors.


## STEP 7: Compile LinuxCNC

```
$ make
```


## STEP 8: Set permissions

As we will be running a real machine, we need to setup the required permissions.

```
$ sudo make setuid
```


## STEP 9: Run LinuxCNC

LinuxCNC should now be ready to use as run-in-place (RIP).

```
$ . ./scripts/rip-environment
$ linuxcnc
```

Linuxcnc should load and be able to open Axis or other non-Qt user interfaces. A RT-Preempt kernel needs to be installed before being able to run a real machine. 

