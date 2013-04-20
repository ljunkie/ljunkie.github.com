---
title: 'Ubuntu 10.04 Lucid 5.1 HDMI Audio (Intel G45) &#8211; Boxee 5.1 HDMI Audio'
author: rob
layout: post
categories:
  - Linux
  - Ubuntu
---
# 

First of all. This is for a Asus P5Q-EM motherboard with Intel G45 onboard. This doc could work for others, but of course I have no clue!

You can verify the hardware first by running :  [aplay -l][1]

 [1]: http://alsa.opensrc.org/index.php/Speaker-test "aplay -l"

```
 aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: Intel [HDA Intel], device 0: ALC1200 Analog [ALC1200 Analog]
Subdevices: 1/1
Subdevice #0: subdevice #0
card 0: Intel [HDA Intel], device 1: ALC1200 Digital [ALC1200 Digital]
Subdevices: 1/1
Subdevice #0: subdevice #0
card 0: Intel [HDA Intel], device 3: INTEL HDMI 0 [INTEL HDMI 0]
Subdevices: 0/1
Subdevice #0: subdevice #0
```

<!-- more -->

You’ll notice card 0 device 3 is the Intel HDMI Audio.. If you don’t have the Intel, look for HDMI and figure out the card and device id for the HDMI out.

A way to test the audio from the card is available by running this speaker-test on the console.

``` 
speaker-test -Dhdmi:Intel -c6 -twavq
```

If that doesn’t work, maybe you don’ thave the intel card, so the -D would not work for your. You can try (replace 0,3 with whatever card,device you have from the output above).

```
speaker-test -Dhw:0,3 -c6 -twavq
```

### The Howto Start for getting 5.1 HDMI Audio in Ubuntu 10.04 ###

1) Remove  PULSEAUDIO (replace with ALSA)

```
sudo apt-get purge pulseaudio gstreamer0.10-pulseaudio
```

```
sudo apt-get autoremove
sudo apt-get install alsa-base alsa-tools alsa-tools-gui alsa-utils alsa-oss linux-sound-base alsamixergui
sudo apt-get install esound esound-clients esound-common gnome-alsamixer
```

2) Reboot

3) Set default audio card (HDMI)  *this step I don’t think does much.. but I did it and my system works… so why not

```
gstreamer-properties
```

4) THE MOST IMPORTANT STEP.! This will set the HDMI out as the default card so Boxee will work properly with your audio

Create an ~/.asoundrc file for the user you login with when you run Boxee

```
$ cat &gt; ~/.asoundrc

#START
pcm.dmixer {
type dmix
ipc_key 1024
ipc_key_add_uid false
ipc_perm 0660
slave  {
pcm "hw:0,3"
rate 48000
channels 2
format S32_LE
period_time 0
period_size 1024
buffer_time 0
buffer_size 4096
}
}
pcm.!default {
type plug
slave.pcm "dmixer"
}
# END
```

* `ctrl-d` to end the cat

5) Disable HARDWARE Acceleration for Video. Otherwise [Boxee][2] will just crash on any local video playback.

 [2]: http://www.boxee.tv/ "Boxee"

- run [Boxee][2]  
- go to – settings -> media -> advanced  
- uncheck — ‘enable hardware assisited decoding when possible’  
- Render Method: Software (or Auto might work)

6) Sound Config Setup

- run [Boxee][2]  
- go to – settings -> system -> Audio Hardware

- Audio Output: Digital
- DTS Capable Reciever: checked
- Dolby Digital (AC3) Reciever: checked  
- Audio output device: default
- Passthrough output device: default`  
- Downmix multichannel audio to stereo: UNCHECK


** NOTE: I will post how to get [BOXEE ][2]installed soon since that was also a pain in the ass with 10.04 release.

