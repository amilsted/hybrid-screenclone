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

# Changes from original

This is not the original version, but a patched version designed so that you can run more
one copy of screenclone on the same screen.  I've added the ability to specify the x_offset (-t)
and the y_offset (-u) where the screen will copied to.  I personally use this with a Thinkpad
W530 when docked to the Mini dock III plus.  The display ports on the dock are wired directly
to the nvidia chipset so it's not available to the intel chip I use while undocked.

By using a combination of the virtual crtc patch and screenclone, I can create a dual screen 
display.  Here is the general script I use:

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
    #activate external monitor and set options for it
    xrandr --output LVDS1 --auto --output VIRTUAL1 --mode $mode --right-of LVDS1
    #run screenclone in optirun. this way the NVIDIA card will automatically start and shut down. kill with ctrl+c
    echo  $mode
    optirun screenclone -s $DISPLAY -d :8 -x 0 &
    optirun screenclone -s $DISPLAY -d :8 -x 1 -t 1920
    #deactivate monitor after screenclone is killed (kill this script with ctrl+C)
    pkill optirun
    xrandr --output VIRTUAL1 --off

