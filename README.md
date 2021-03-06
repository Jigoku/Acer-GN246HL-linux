# Acer-GN246HL-linux
Some information about using an Acer GN246HL monitor on Linux with nVidia lightboost.
I have spent hours searching the web in order to setup this monitor within Linux, and hope that i can save some other peoples time by writing up my findings, and solutions.

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

## Custom Modelines relating to lightboost

Skip this part if you want to, as it's not neccesary to configure the monitor in X11 if you wish to use the EDID file contained in this repository.

As for the previous paragraphs, with the link to custom modelines; as example for 120hz;
```
Modeline "1920x1080_120lb" 286.7 1920 1968 2000 2080 1080 1083 1088 1149 +HSync -VSync
```

This is an invalid modeline, hence you have to allow non edid modes. The reason for this, is that the last paramater for the vertical trace, should in fact be 1144. However, for the sake of enabling lightboost, this needs to be within the range of 1149-1180. I'm not fluent in the reasons for this, but thought i'd mention it for the sake of using `cvt` to generate a modeline.

```
$ cvt -r 1920 1080 120
# 1920x1080 119.88 Hz (CVT) hsync: 137.14 kHz; pclk: 285.25 MHz
Modeline "1920x1080R"  285.25  1920 1968 2000 2080  1080 1083 1088 1144 +hsync -vsync
```
As you can see, for 1920x1080 @ 120hz, this tells us the last value is 1144. When you increase this to 1149 (as to enable lightboost), the pixel clock has to also increase to reflect that. So 285.25 becomes 286.7. 

## Setting up X11

Here's how to configure this monitor on X11 in Linux (possibly with other *nixes too).

You can download my EDID.bin from [here](https://github.com/Jigoku/Acer-GN246HL-linux/blob/master/EDID.bin), just place this at */etc/X11/EDID.bin*, then use the following xorg related config to load it;

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
    Option         "NoLogo" "1"
    Option         "CustomEDID" "DVI-D-0:/etc/X11/EDID.bin"
EndSection
```

Note the last few lines; where it states *DVI-D-0*, you may have to change this to whatever your monitor is labelled as, this will depend on the type of cable you are using, but should be at least a DVI name when wanting to use lightboost.

Make sure you don't have any other *.conf files in /etc/X11/xorg.conf.d/ and neither an /etc/X11/xorg.conf, as this single nvidia related file is all that is needed to get lightboost working with nvidia's proprietary driver.
You should now be able to launch nvidia-settings, and see these refresh rates:

* 60hz
* 100hz
* 120hz
* 144hz

Note, that only 100hz and 120hz are supported by lightboost.

Run ` glxgears` to confirm this is working;

Eg: 
```
602 frames in 5.0 seconds = 120.384 FPS
```

## ddcutil

See the problem here: http://www.ddcutil.com/nvidia/

Create */etc/modprobe.d/nvidia-ddcutil-fix.conf* containing:
```
options nvidia NVreg_RegistryDwords=RMUseSwI2c=0x01;RMI2cSpeed=100
```

This should allow detection of the ddc capabilities
```
# ddcutil detect
Display 1
   I2C bus:             /dev/i2c-7
   Supports DDC:        true
   EDID synopsis:
      Mfg id:           ACR
      Model:            GN246HL
      Serial number:    LW3EE0058533
      Manufacture year: 2017
      EDID version:     1.3
   VCP version:         2.1
```

#### Controlling the lightboost value over ddc

The VCP code 0xfa is responsible for setting the lightboost value in increments of 10.
``` 
VCP code 0xfa (Manufacturer Specific         ): mh=0x00, ml=0x0a, sh=0x00, sl=0x00, max value =    10, cur value =     0
```

* set lightboost to maximum
```
$ ddcutil setvcp 0xfa 10
```
* set lightboost to 50%
```
$ ddcutil setvcp 0xfa 5
```

You can extract the values for the ddc lightboost setting using these comands:

* current value 
```
$ ddcutil getvcp 0xfa -t | awk {'print $4'}
```

* maximum value
```
$ ddcutil getvcp 0xfa -t | awk {'print $5'}

```

#### ddc features not advertised as capabilities

I am tempted to believe 0xe0, 0xe1, 0xe7, 0xe9 and 0xeb have something to do with configuring lightboost, i may be wrong here, but most of the values don't update when you try to change them. One thing that is true, is that 0xfa is responsible for setting the brightness of lightboost (the setting that is also on the OSD).
The features labelled as "phase" could have something to do with configuring the strobing, but these values cannot be changed either.

If anyone else with this monitor can crack this and finds out anything else...

```
   Feature x03 - Soft controls
   Feature x0b - Color temperature increment
   Feature x0c - Color temperature request
   Feature x0e - Clock
   Feature x1e - Auto setup
   Feature x1f - Auto color setup
   Feature x20 - Horizontal Position (Phase)
   Feature x30 - Vertical Position (Phase)
   Feature x3e - Clock phase
   Feature x62 - Audio speaker volume
   Feature xa8 - Unknown feature
   Feature xc0 - Display usage time
   Feature xca - OSD
   Feature xcb - Unknown feature
   Feature xdc - Display Mode
   Feature xe0 - Manufacturer Specific
   Feature xe1 - Manufacturer Specific
   Feature xe7 - Manufacturer Specific
   Feature xe9 - Manufacturer Specific
   Feature xeb - Manufacturer Specific
   Feature xfa - Manufacturer Specific
```

## Firefox tweaking

Now there are some other tweaks you will have to make. One notable one is firefox. I was getting horrific blur/ghosting when scrolling any web page, and found a solution by forcing the refresh rate.
Head to "about:config" in your browser. Search for "layout.frame_rate". 

Force this setting to whatever your chosen refresh rate is. In my case, as i wanted to use the lightboost feature with 120hz, i set this value to 120.

After restarting firefox, this should give you the crystal clear sharpness that is present on Windows when scrolling documents. This setting over ride should also make the ghosting/blur tests available at https://www.testufo.com/ work just as perfectly as in Windows. Despite vsync not being supported, you may get a slight stutter once in a while, but over all this should look sharp without any ghosting or blurring.

the default value of "-1" is supposed to represent vsync, but for some reason this does not sync to the current refresh rate under Linux, i believe this may be a bug within firefox. But for now, forcing the refresh rate on this setting seems to be acceptable for ghosting-free and blur-free scrolling.

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
