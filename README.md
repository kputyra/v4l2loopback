v4l2loopback - a kernel module to create V4L2 loopback devices
==============================================================

this module allows you to create "virtual video devices"
normal (v4l2) applications will read these devices as if they were ordinary
video devices, but the video will not be read from e.g. a capture card but
instead it is generated by another application.
this allows you for instance to apply apply some nifty video effects on your
Skype video...
it also allows some more serious things (e.g. I've been using it to add
streaming capabilities to an application by the means of hooking GStreamer into
the loopback devices).

# NEWS
to get the main features of each new release, see the NEWS file.
you could also have a look at the ChangeLog (which gets automatically generated and might
only be of limited use...

# ISSUES
for current issues, checkout https://github.com/umlaeute/v4l2loopback/issues
please use the issue-tracker for reporting any problems

# DEPENDENCIES
the v4l2loopback module is a *kernel module*.
in order to build it, you *must have* the kernel headers installed that match
the linux kernel with which you want to use the module (in most cases this will
be the kernel that you are currently running).
please note, that kernel headers and kernel image must have *exactly the same* version.
for example, `3.18.0-trunk-rpi` is a different version that `3.18.7-v7+`, even though
the first few number are the same.
(modules will be incompatible if the versions don't match. if you are lucky, the module will
simply refuse to load. if you are unlucky, your computer will spit in your eye or do worse.)

# BUILD
to build the kernel module run:

    $ make

this should give you a file named "v4l2loopback.ko", which is the kernel module

# INSTALL
to install the module run "make install" (you might have to be 'root' to have
all necessary permissions to install the module).

if your system has "sudo", do:

    $ make && sudo make install
    $ sudo depmod -a

if your system lacks "sudo", do:

    $ make
    $ su
    (enter root password)
    # make install
    # depmod -a
    # exit


(The `depmod -a` call will re-calculate module dependencies, in order to
automatically load additional kernel modules required by v4l2loopback.
The call may not be necessary on modern systems.)

# RUN
load the v4l2loopback module as root :

    # modprobe v4l2loopback

using sudo use:

    $ sudo modprobe v4l2loopback

this will create an additional video-device, e.g. /dev/video0 (the number
depends on whether you already had video devices on your system), which can be
fed by various programs.
tested feeders:
- GStreamer-0.10: using the  "v4l2sink" element
- Gem(>=0.93) using the "recordV4L2" plugin
in theory most programs capable of _writing to_ a v4l2 device should work.

the data sent to the v4l2loopback device can then be read by any v4l2-capable
application.

you can find a number of scenarios on the wiki at
	http://github.com/umlaeute/v4l2loopback/wiki

# OPTIONS
if you need several independent loopback devices, you can pass the "devices"
option, when loading the module; e.g.

    # modprobe v4l2loopback devices=4

will give you 4 loopback devices (e.g. `/dev/video1` ... `/dev/video5`)
you can also specify the device IDs manually; e.g.

    # modprobe v4l2loopback video_nr=3,4,7

will create 3 devices (`/dev/video3`, `/dev/video4` & `/dev/video7`)

    # modprobe v4l2loopback video_nr=3,4,7 card_label="device number 3","the number four","the last one"

will create 3 devices with the card names passed as the second parameter:
- `/dev/video3` -> *device number 3*
- `/dev/video4` -> *the number four*
- `/dev/video7` -> *the last one*

# ATTRIBUTES
you can set and/or query some per-device attributes via sysfs, in a human
readable format. see `/sys/devices/virtual/video4linux/video*/`

also there are some V4L2 controls that you can list with

    $ v4l2-ctl -d /dev/video1 -l

- `keep_format(0/1)`: while set to 1, once negotiated format will be fixed forever,
                  until the setting is set back to 0
- `sustain_framerate(0/1)`: if set to 1, nominal device fps will be ensured by means
                        of frame duplication when needed
- `timeout(integer)`: if >0, will cause a timeout picture (a null frame, by default)
                  to be displayed after (value) msecs of missing input
- `timeout_image_io(0/1)`: if set to 1, the next opener will write to timeout frame
                       buffer

# FORCING FPS

    $ v4l2loopback-ctl set-fps 25 /dev/video0

or

    $ echo '@100' | sudo tee /sys/devices/virtual/video4linux/video0/format

# FORCING A GSTREAMER (0.10) CAPS

    $ v4l2loopback-ctl set-caps "video/x-raw-yuv, width=640, height=480" /dev/video0

# SETTING STREAM TIMEOUT
~~~
$ v4l2-ctl -d /dev/video0 -c timeout=3000
(will output null frames by default)
$ v4l2loopback-ctl set-timeout-image service-unavailable.png /dev/video0
this currently requires GStreamer 0.10 installed
~~~

# KERNELs
the original module has been developed for linux-2.6.28;
i don't have a system with such an old kernel anymore, so i don't know whether
it still works.
further development has been done mainly on linux-2.6.32 and linux-2.6.35, with
newer kernels being continually tested as they enter Debian.

support:
- >= <kbd>4.0.0</kbd>		should work
- >= <kbd>3.0.0</kbd>		might work
- << <kbd>3.0.0</kbd>		may work (has not been tested in ages)
- <= <kbd>2.6.27</kbd>		will definitely NOT work

# DISTRIBUTIONS
v4l2loopack is now (since 2010-10-13) available as a Debian-package.
https://packages.debian.org/source/stable/v4l2loopback

this means, that it is also part of Debian-derived distributions, including
Ubuntu (starting with natty).
the most convenient way is to install the package "v4l2loopback-dkms":

    # aptitude install v4l2loopback-dkms

this should automatically build and install the module for your current kernel
(provided you have the matching kernel-headers installed).
another option is to install the "v4l2loopback-source" package.
in this case you should be able to simply do (as root):

    # aptitude install v4l2loopback-source module-assistant
    # module-assistant auto-install v4l2loopback-source

# DOWNLOAD
the most up-to-date version of this module can be found at
http://github.com/umlaeute/v4l2loopback/.

# LICENSE/COPYING

- Copyright (c) 2010-2016 IOhannes m zmoelnig
- Copyright (c) 2016 Gavin Qiu
- Copyright (c) 2016 George Chriss
- Copyright (c) 2014-2015 Tasos Sahanidis
- Copyright (c) 2012-2015 Yusuke Ohshima
- Copyright (c) 2015 Kurt Kiefer
- Copyright (c) 2015 Michel Promonet
- Copyright (c) 2015 Paul Brook
- Copyright (c) 2015 Tom Zerucha
- Copyright (c) 2013 Aidan Thornton
- Copyright (c) 2013 Anatolij Gustschin
- Copyright (c) 2012 Ted Mielczarek
- Copyright (c) 2012 Anton Novikov
- Copyright (c) 2011 Stefan Diewald
- Copyright (c) 2010 Scott Maines
- Copyright (c) 2009 Gorinich Zmey
- Copyright (c) 2005-2009 Vasily Levin

    This package is free software; you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation; either version 2 of the License, or
    (at your option) any later version.

    This package is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program. If not, see <http://www.gnu.org/licenses/>.