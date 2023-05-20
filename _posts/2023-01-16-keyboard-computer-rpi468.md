---
layout: post
title: "Raspberry Pi 468: a full computer in a mechanical keyboard"
author: "Paul Vinh Phan"
categories: journal
image: rpi468greyclose.jpg
tags: [raspberry pi, rpi, 400, mechanical keyboard, tada68, 65%]
---

{:centeralign: style="text-align: center;"}

Table of Contents:
* TOC
{:toc}

# Intro
I'm someone who appreciates a tactile keyboard experience.
When keystrokes have a positive, discrete feel, it provides a passive confirmation to my hands that my keypresses have had the intended effect.
This makes typing and writing code both more efficient and more enjoyable.

I'm also someone who loves the ethos around the Raspberry Pi single board computer -- a small, inexpensive computer put into the hands of as many people as possible.
As well as their iconic credit card sized form factor, the company behind Raspberry Pi also made the Raspberry Pi 400.
The 'Pi 400' is a fully featured Raspberry Pi built into a slim keyboard, featuring most of the IO ports of the 'B+' form factor.
The density of utility of the Pi 400 was something I just had to have, even if I didn't have an immediate need for it.

![](assets/img/2023-01-16-keyboard-computer-rpi468/pi400.jpg)
{: centeralign }
The official Raspberry Pi 400 keyboard computer ([source](https://www.raspberrypi.com/products/raspberry-pi-400/)).
{: centeralign }

After a somewhat impulsive purchase from my local Micro Center, I went home with a Pi 400.
It was both incredible and lack luster in the ways I expected: it was a bonafide Raspberry Pi computing platform, and the keyboard was mediocre to type on (mushy and uneven feeling, shallow key travel).
The obvious conclusion that I came to was: why not combine a tactile mechanical keyboard with the main board of the Pi 400?

And so I set to work to combine my trusty (albiet aging) Tada 68 mechanical keyboard with the main board of the Pi 400.
It seemed fitting to combine the names:

Raspberry Pi 400 + Tada 68 = Raspberry Pi 468.

If you're interested in building one of your own, check out the [Design files](#design-files) section below.


# Design goals
I drew inspiration from [Pavlo Khmel on YouTube](https://www.youtube.com/watch?v=TTT5TCiPke4&pp=ygUkcmFzcGJlcnJ5IHBpIDQwMCBtZWNoYW5pY2FsIGtleWJvYXJk&ab_channel=PavloKhmel) who also upgraded the Pi 400 to be integrated with a mechanical keyboard.
However, there were design decisions made in his implementation that I wanted to do differently:
1. His mechanical keyboard was connected to the Pi 400 by running a USB A cable out of the case and then back in.
Functionally, there is nothing wrong with this, but aesthetically I could not abide it -- for my keyboard the connecting cable had to be internally routed.
2. When the mechanical keyboard is connected to the Pi 400, it cannot be used by any other host device such as a desktop PC.
A desirable feature to me is to use the keyboard with a host device while also running the Pi as a headless server (e.g. for interfacing over I2C, GPIO, etc).

The goal for my design was to have a seemless experience in operating in either 'keyboard computer' mode, or 'keyboard only' mode (for a host PC).
To achieve this and also eliminate a USB A cable loop outside of the case, I needed to implement a USB switching circuit board.
Another design constraint I imposed was that the form factor of the keyboard be indistinguishable from any similarly sized mechanical keyboards (65% layout).
After some research, I realized this circuit board did not exist in the form factor I needed, and I'd have to design and build a custom one.

For the mechanical design I settled on a two-part case made of 3D printed material.
I decided on mechanical fastening of the two halves over adhesive for strength and repairability.
I toyed with the idea of a unibody CNC'd aluminum case, but cost and fear of wireless signal degradation took me off this path.
In the second iteration of the case design, I included a CNC'd aluminum heat sink for passive heat dissapation of the Pis CPU and RAM.


# Design files

Since the build process didn't go very cleanly for me, I won't try documenting exactly how to reproduce building this.
For those familiar with SMD soldering and 3D printing though, the images in the gallery and these github repos should provide direction.
One caveat: the case model files are for the updated version of the case that has a cutout for a heat sink.
Aside from this it's basically the same case.

Github:
- Model files for case (STLs + SolidWorks): [https://github.com/pvphan/rpi468](https://github.com/pvphan/rpi468)
- PCB design files (KiCAD): [https://github.com/pvphan/rpi468-pcb](https://github.com/pvphan/rpi468-pcb)

PCB projects on OSHPark:
- [USB switch board](https://oshpark.com/shared_projects/PfGfJg1X)
- [Mini USB pad to JST](https://oshpark.com/shared_projects/LUSGZfi8)

I used KiCAD for the USB switching circuit PCB design and SolidWorks for mechanical design of the case.
PCB and stencil fab was done through OSHPark and OSHStencils.
Electrical components were ordered off DigiKey and mechanical parts from McMaster.

For surface mount device (SMD) soldering, I tried a reflow oven but had poor results.
I found more success using a combination of hot plate from the bottom and hot air from the top simultaneously.


# Challenges
The concept and design of this project were thankfully very straightforward.
The most significant challenges were at the execution level:
- Designing the USB switcher circuit with my novice level of circuit design skill.
- Soldering the final USB switcher circuit with SMD components.
- Desoldering and soldering the USB port on the assembled keyboard PCB
- Designing the tighter tolerance features around the Pi 400 PCB: mounting holes, snap clips


# Final result

![](assets/img/2023-01-16-keyboard-computer-rpi468/PXL_20220828_194531996.jpg)
{: centeralign }
Running Raspberry Pi OS on the RPi 468 in 'keyboard computer' mode at my desk.
{: centeralign }

Overall I'm quite happy with the result.
Generally I use it most as a regular keyboard, especially when I travel.
Since my work spaces at the office and at home have their own dedicated keyboard, this one usually stays in my backpack as a backup.

It's strangely comforting to know that I have the low-level hardware capability of a Raspberry Pi with me at all times without any additional weight or backpack space being taken up.

{% include image-gallery.html folder="/assets/img/2023-01-16-keyboard-computer-rpi468" %}
