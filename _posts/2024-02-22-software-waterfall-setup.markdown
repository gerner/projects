---
layout: post
title:  "Software Waterfall Setup for my Radio Operation"
date:   2024-02-22 10:26:00 -0800
categories: radio
---

![Hero shot of SDR++ running my plugins]({{site.baseurl}}/assets/images/sdrpp_rigsync_spots.png)

## A Radio Contact Console

[SDR++](sdrpp) has a better waterfall than my radio, and I can overlay helpful
visuals on top of it like the bandplan or spots. So I want to synchronize my
radio and SDR++. I can rely on SDR++ to provide information about band
activity, while I make contacts on my radio. I can tune from my radio or from
SDR++ and they both always stay in sync. I’m pulling in spots from
[pota.app](potaapp) or from DX Cluster (via [hamqth](hamqth)). Using this
setup, I can quickly navigate to frequencies of interest and make contacts on
my radio.

## SDR++ Plugins

To get this setup working, I had to make two plugin modules for SDR++:

 * [RigSync](rigsync) synchronizes my rig and SDR++
 * [Spots](spots) visualizes spots on the SDR++ waterfall

RigSync, synchronizes my rig and SDR++ using [Hamlib](hamlib) rigctl. It works
in both directions, so I can tune on either the radio or in SDR++ and the other
one will be tuned to match automatically. This way I can find interesting
signals in SDR++ and then receive or transmit using my radio. And if I wander
down the band on my radio, SDR++ will automatically follow along.

SDR++ actually has a rigctl client plugin currently in development. I wanted to
use that work, but it's missing a lot of the functionality I'm looking for. And
having spoken with the developer a bit, it's not clear that my use case is
exactly what they're trying to support. So I ended up scrapping that plan and
basically starting from scratch.

Spots displays spots on the waterfall, shows details on hover, and lets me tune
directly to spot frequencies. I can see frequencies that are likely to have
people looking for contacts. I can see when they were last spotted as well as
their location.

The Spots plugin listens for external connections to pull in spots. I'm not
sure if this is the best way do it. It does make the plugin a little harder to
use. But I didn't want to code up different spot sources in C++. This way I can
quickly add another spot source with a little bash script. I did write a little
tool in bash that polls Pota.app or HamQTH for spots and feeds them into the
plugin.

## Setup

I'm running Ubuntu Linux, so all of these steps are specific to linux. With a
little persistence you should be able to get things working on Windows. Of
course, there's lots of ham radio software alternatives for Windows.

SDR++ gets its RF information from some radio source. In my case this is an
[SDRPlay RSP1B](sdrplay_rsp1b) software defined radio. This radio has a
proprietary computer interface that requires installing their library and
service to interact with the radio on a computer.

I have an IC-7300 radio and have udev rules that give a friendly `/dev` name to
the tty interface. I might write up that info in a future post.

1. Start SDRPlay service if not already running
```
systemctl start sdrplay
```
2. Turn rig on and plug into computer
3. Start rigctld
```
rigctld -m373 -r /dev/ic7300
```
4. Start SDR++, will start rigctl client and spot server
5. Start spot daemon to feed spots into SDR++ (`--source=hamqth` or `--source=pota`)
```
./spots.sh --source=hamqth
```

SDR++ should now show spots on the waterfall. You can start RigSync ("Start"
button on the left panel somewhere) and any tuning you do in SDR++ will be
reflected on your rig and vice versa.

[sdrpp]: <https://www.sdrpp.org/> "SDR++ visualizes radio signals on a waterfall"
[potaapp]: <https://pota.app> "Pota.app hosts information about parks on the air activations"
[hamqth]: <https://www.hamqth.com> "HamQTH has an API that shares dxcluster spots"
[rigsync]: <https://github.com/gerner/sdrpp-rigsync> "SDR++ RigSync plugin synchronizes frequencies for a rig and SDR++"
[spots]: <https://github.com/gerner/sdrpp-spots> "SDR++ Spots plugin adds spots to an SDR++ waterfall"
[hamlib]: <https://hamlib.github.io/> "Hamlib is a library and tools that provides radio control"
[sdrplay_rsp1b]: <https://www.sdrplay.com/rsp1b/> "SDRPlay RSP1B software defined radio"
