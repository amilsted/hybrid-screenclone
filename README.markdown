# hybrid-screenclone

This is a reimplementation of [hybrid-windump][hybrid-windump] with the
opposite use-case: doing all rendering using the integrated intel card and
using the additional card just to get more outputs (e.g. a triple-head with
ThinkPad T420). As such, it uses the DAMAGE extension to avoid unnecessary
redraws and the RECORD extension to capture mouse movements, which are
translated to mouse movements on the destination X server.

For this to work correctly, an additional virtual Xinerama screen must be
available. To get one, see my [virtual CRTC for intel][patch] patch.

[hybrid-windump]: https://github.com/harp1n/hybrid-windump
[patch]: https://github.com/liskin/patches/blob/master/hacks/xserver-xorg-video-intel-2.18.0_virtual_crtc.patch

---------------------------

# Changes from original

This version clones two screens to the destination display. The user specifies
an offset for the second screen. This gives me a three-headed setup on my
W520 (optimus) using a script like this:

	#!/bin/bash
	#change values below to your defaults
	x=1920
	y=1080
	if [ -n "$1" ]
	then
		x=$1
	fi
	if [ -n "$2" ]
	then
		y=$2
	fi
	#generate a modeline from x/y
	modeline=`cvt $x $y | sed "1d" | sed 's/Modeline //'`
	mode=`echo $modeline | sed 's/ .*//'`

	#create the mode and ignore xrandr error if the mode is already there
	xrandr --newmode $modeline &> /dev/null 2>&1
	#add the mode to the other display
	xrandr --addmode VIRTUAL1 $mode
	xrandr --addmode VIRTUAL2 $mode
	#activate external monitor and set options for it
	xrandr --output LVDS1 --auto --output VIRTUAL1 --mode $mode --left-of LVDS1 --output VIRTUAL2 --mode $mode --left-of VIRTUAL1
	#run screenclone in optirun. this way the NVIDIA card will automatically start and shut down. kill with ctrl+c
	optirun /home/ash/pkgs/screenclone-git/src/screenclone/screenclone -s $DISPLAY -d :8 -x 2 -z 1 -t 1920
	#deactivate monitor after screenclone is killed (kill this script with ctrl+C)
	xrandr --output VIRTUAL1 --off --output VIRTUAL2 --off
