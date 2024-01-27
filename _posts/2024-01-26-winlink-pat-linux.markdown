---
layout: post
title:  "Winlink over My HT under Linux with Pat and Digirig"
date:   2024-01-25 11:54:33 -0800
categories: jekyll update
---

![Hero shot of my Yaesu FT-65R, Digirig and laptop ready for Winlink]({{site.baseurl}}/assets/images/ft65_digirig.png)

These are just some quick notes about how I got [Winlink][winlink] to work
under Linux using [Pat][pat], [hamlib rigctld][rigctld] over my Yaesu FT-65R
using a [Digirig][digirig] and [VaraFM][varafm]. I used the
[West Seattle Amateur Radio Club][wsarc]
[Winlink gateway][wsarc_winlinkgateway].

## TL;DR

You need:
 * A TNC (e.g. digirig) connecting your HT to your computer
 * HT tuned to the Winlink gateway frequency
 * rigctld interfaced with the TNC
 * VaraFM running under Wine, configured with the TNC's sound devices and COM
   port for PTT
 * Pat configured to talk to rigctld and VaraFM

Here are the commands:

Start rigctld (it'll keep running)

```bash
rigctld -p /dev/ttyUSB0 -P RTS
```

Start VaraFM (it'll keep running)

```bash
WINEPREFIX=~/.wine-winlink wine '/home/nick/.wine-winlink/drive_c/VARA FM/VARAFM.exe'
```

Send/receive mail with Pat (once configured properly)

```bash
pat connect 'varafm:///W7AW-10?freq=439.850'
```

Gotchas I ran into with solutions:
 * Pat does not seem to support HTs without remote rig control: run rigctld
   even though it won't be doing anything
 * COM port mapping: see KM4AC's video on
   [mapping linux device to COM port][km4ac_comport]
 * ALSA devices not showing up in Wine: don't run pavucontrol or other app that
   grabs devices
 * Poor signal strength or no connection
   * make sure your HT is upright and has reasonable connection just as a radio
   * check the audio level on the HT and/or computer sound device
   * run a VaraFM calibration (under VaraFM > Settings > Sound Card... > Auto
     Tune)

## Notes

### VaraFM

![VaraFM mediocre calibratio results]({{site.baseurl}}/assets/images/varafm_calibration.png)

I followed some guide on installing it under linux, but basically download and
run the installer under wine. I have a WINEPREFIX set up just for Winlink.
Maybe that'll grow into a general purpose ham Wine setup since it seems like
tons of good ham radio apps
are Windows only.

I made sure to set up my soundcard with the USB soundcard options for input and
output. This might be linux specific, but for some reason pavucontrol (and
probably other apps that grab sound devices under linux) prevent sound devices
from showing up in wine! So those apps cannot be running while VaraFM is
running.

And I chose COM 10 with RTS as the PTT configuration. That COM port maps to my
digirig as per this [COM port mapping tutorial][km4ac_comport]. It's not clear
if I need that since rigctld could control PTT. I'm not sure if VARA could just
not do that and leave it to Pat/rigctld.

Also linux specific, I have `sound=alsa` from [winetricks][winetricks_alsa].
Otherwise it'll use pulseaudio (or maybe pipewire if you're using a linux OS
from the last few years). I hear pulseaudio causes the audio to be jittery or
something. It also hides the audio devices from Wine and just lists
"Pulseaudio". So then you'd have to set those up...  somehow else?

The popup asking for $65 during transmission is kinda irritating. But it is
nice of EA5HVK to provide it. Maybe you and I ought to buy licenses...

```bash
WINEPREFIX=~/.wine-winlink wine '/home/nick/.wine-winlink/drive_c/VARA FM/VARAFM.exe'
```

### Digirig

This was really easy. I just got the digirig. It arrived today. It's tiny,
smaller than I thought, about the size of a fat thumb. I also got the cables to
connect to my FT-65 and my Baofeng GT-5R.

Just plug the digirig into the computer via USB and the HT into the digirig via
the HT-specific cable(s) that digirig sells you. In my case it's only using the
"Audio" port on the digirig. I guess the other one is for CAT control if you've
got a rig with full-on remote VFO (and other) controls. But if it's got that,
then it might have a built in TNC, so why do you need a digirig?

### Hamlib/rigctld

Again, pretty easy. In my setup I don't think it's actually doing anything
since my Pat config says not to use PTT control. I guess VaraFM is doing PTT
control? This is a bit of a Rube Goldberg contraption.

```bash
rigctld -p /dev/ttyUSB0 -P RTS

```

`/dev/ttyUSB0` is where the digirig shows up.

`-P RTS` is the PTT type: RTS uses one of the rings on the audio jack I think.
But that's just how PTT control works on the FT-65. That might work for other
HTs too.

### HT: Yaesu FT-65R

I just turned my HT on and tuned into the W7AW-10 Winlink Gateway at 439.850
MHz. Using my HF rig Pat can control the VFO, which is convenient, but my HT is
simple and has no remote VFO control. So it just sits there waiting for VaraFM
and the digirig to hit the PTT and send/recv audio.

I did have my HT fall over at one point and VaraFM could no long connect to the
gateway. So the connection isn't superb.

I think you can tune the audio level if you hold the monitor button down and
watch the audio input VU meter in VaraFM. I assume you want it high, but not in
the red. It seemed fairly touchy to me. VaraFM calibration can also help you
dial this in. That's the "Auto Tune" option under VaraFM > Sound Card...


### Pat

![Pat CLI output showing a successful connect and message send]({{site.baseurl}}/assets/images/pat_connect.png)

Last but not least is my linux-based Winlink client, Pat. Here's the relevant
sections of my config that set up my HT as a Pat rig and configures Pat's
VaraFM protocol.  Obviously I have a lot more config, but almost all of that is
default stuff or easy stuff like my callsign/password, locator, etc.

```json
{
  "hamlib_rigs": {
      "digirig": {"address": "localhost:4532", "network":"tcp"}
  },
  "varafm": {
    "addr": "localhost:8300",
    "bandwidth": 0,
    "rig": "digirig",
    "ptt_ctrl": false
  }
}
```

## Thank You

I could not have done this if Brendan Kitts, KE7TNM, and Ryan Weber, NN7M, from
the West Seattle Amateur Radio Club had not set up the W7AW-10 Winlink gateway.
They also both gave me a lot of feedback and encouragement as I was trying to
make connections.

I hope this helps you, even if "you" is really "me".

[winlink]: <https://winlink.org/> "Winlink provides an email gateway for amateur radio signals"
[pat]: <https://getpat.io/> "Pat is a Winlink client for linux"
[digirig]: <https://digirig.net/> "Digirig is a USB soundcard to connect radios without a built in TNC to a computer"
[varafm]: <https://winlink.org/tags/varafm> "VaraFM is a protocol for sending data over RF and the associated app to transcode from data into audio signals (aka modem) over RF"
[rigctld]: <https://hamlib.sourceforge.net/html/rigctld.1.html> "Hamlib rigctld is a linux app that can control radio rigs and provide access to other apps"
[wsarc]: <https://w7aw.org"> "West Seattle Amateur Radio Club"
[wsarc_winlinkgateway]: <http://www.appliedaisystems.com/w7aw_monitor.html> "WSARC Winlink gateway monitoring page"
[km4ac_comport]: <https://www.youtube.com/watch?v=iYYRTu6myRc> "KM4AC shows setting up a COM port to linux device mapping for Wine"
[winetricks_alsa]: <https://askubuntu.com/questions/77210/how-to-change-the-default-audio-in-wine-to-alsa-only> "Info on using winetricks to configure the audio source"
