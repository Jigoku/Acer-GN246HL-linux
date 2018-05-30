# Acer-GN246HL-linux
Some information about using an Acer GN246HL monitor on Linux with nVidia lightboost.

## Background
Oh boy, was this a hassle. 

The Acer GN246HL monitor is advertised as here: https://www.acer.com/ac/en/US/content/model/UM.FG6AA.B01

The features that stood out to me:

* 1 ms Response Time
* upto 144 Hz refresh rate
* nVidia lightboost

I ended up getting one of these on a whim, as my previous monitor decided to die.

Initially, i had some problems figuring out lightboost. I could not seem to enable it, and the OSD display within the monitor had this option greyed out. 

After countless hours of searching online, i came across some useful information from here: http://forums3.armagetronad.net/viewtopic.php?f=1&t=23173
Following this guide enabled lightboost on my monitor and also gave access to 100 / 120 / 144hz refresh rates within `nvidia-settings`.

Now whilst this worked, i had also connected a Windows10 HDD, to see how it "should" be performing as a monitor. I followed the guides here relating to windows https://www.blurbusters.com/zero-motion-blur/lightboost/
(see "Alternate LightBoost HOWTO #3: ToastyX Custom Resolution Utility")

Once i had followed this, i decided to dump the EDID.bin from within Windows after configuring lightboost so that i could use it within X11 on Linux, so things are "like for like". 

## Setting up X11

Here's how to configure this monitor on X11 in Linux (possibly with other *nixes too).

You can download my EDID.bin from [here](https://github.com/Jigoku/Acer-GN246HL-linux/blob/master/files/EDID.bin), just place this at */etc/X11/EDID.bin*, then use the following xorg related config to load it;

*/etc/X11/xorg.conf.d/10-nvidia.conf*
```
Section "Screen"
    Identifier     "Screen0"
    Device         "Device0"
    Monitor        "Monitor0"
    DefaultDepth    24
    Option         "metamodes" "1920x1080_120 +0+0 { ForceFullCompositionPipeline = On }" 
    Option         "TripleBuffer" "on" 
    Option         "AllowIndirectGLXProtocol" "off" 
    SubSection     "Display"
        Depth       24
    EndSubSection
EndSection

Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName	   "GeForce GTX 1060 3GB"
    Option         "NoLogo" "1"
    Option         "CustomEDID" "DVI-D-0:/etc/X11/EDID.bin"
EndSection
```

Note the last few lines; where it states *DVI-D-0*, you may have to change this to whatever your monitor is labelled as, this will depend on the type of cable you are using, but should be at least a DVI name when wanting to use lightboost. You may be better off at first, using `nvidia-xconfig` to generate a config file to find the relevant information for the "Device" section, including your graphics card model etc, as this will probably differ. Make sure you remove */etc/X11/xorg.conf* afterwards.

Make sure you don't have any other xorg.conf files, as this is all that is needed with nvidia's driver.
You should now be able to launch nvidia-settings, and see these refresh rates:

* 100hz
* 120hz
* 144hz

Note, that only 100hz and 120hz are supported by lightboost.

Run ` glxgears` to confirm this is working;

Eg: 
```
602 frames in 5.0 seconds = 120.384 FPS
```

## Firefox tweaking

Now there are some other tweaks you will have to make. One notable one is firefox. I was getting horrific blur/ghosting when scrolling any web page, and found a solution by forcing the refresh rate.
Head to "about:config" in your browser. Search for "layout.frame_rate". 

Force this setting to whatever your chosen refresh rate is. In my case, as i wanted to use the lightboost feature with 120hz, i set this value to 120.
This should give you the crystal clear sharpness that is present on Windows when scrolling documents. This setting over ride should also make the ghosting/blur tests available at https://www.testufo.com/ work just as perfectly as in Windows. Despite vsync not being supported, you may get a slight stutter once in a while, but over all this should look sharp without any ghosting or blurring.

## KDE5

One thing specific to KDE5 is that you can also force the framerate to match the refresh rate, by editing *~/.config/kwinrc*

In the [Compositing] section, add/adjust these values;

```
RefreshRate=120
MaxFPS=120

```

This will improve everything within KDE5 whilst using the plasma desktop, so that it actually uses 120hz. I typically run AwesomeWM, along with xcompmgr, but have not found a solution to this yet. I am also unaware of any other window managers/compositors that can be forced to render at anything higher than 60fps.


## TODO 
When i find more information, for any troublesome software relating to high refresh rates, i will try to update this and add them here.
