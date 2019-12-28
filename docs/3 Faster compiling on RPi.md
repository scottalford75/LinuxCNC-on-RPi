---
title: 3. Faster compiling on RPi
nav: true
---

# Faster compiling on RPi

Compiling kernels and LinuxCNC on the RPi takes quite some time. This can be speed up by using Distcc to share the compiling workload to other ARM or x86_64 computers. The setup of Distcc is well documented on the [Arch Linux Arm Distributed Compiling](https://archlinuxarm.org/wiki/Distributed_Compiling) page but there are a couple of points that should be highlighted.

1. There is an AUR package called [distccd-alarm](https://aur.archlinux.org/pkgbase/distccd-alarm/) that takes away the work of installing on a x86_64 Arch Linux or Manjaro computer. It installs the cross compiling tools that are needed and sets everything up. The only thing that needs customisation is the DISTCC_HOSTS IP address range to suit your local network. Enable and start the appropriate distccd services for the ARM architectures that you wish to compile for.

2. The biggest catch was figuring out that the gcc version of the Master system and that in the x-tools installed on the Client system must be the same. At the time of writing the x-tools pre-built crosstool-ng toolchains is gcc 8.3.0. My Manjaro system had the latest version and required the gcc and gcc-libs packages to be downgraded to 8.3.0. Once this was done everying worked as it should.

Building packages with Arch Linux / Manjaro can use distcc by adjusting the `makepkg.conf` configuration file. If not using makepkg and using `make` directly the following commands will invoke distcc during the build process.

```
$ export DISTCC_HOSTS="<IP>:<PORT>"
$ make -j8 CC="distcc gcc" CXX="distcc g++"
```
