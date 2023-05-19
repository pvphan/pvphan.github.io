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
The 'Pi 400' is a fully featured Raspberry Pi built into a slim keyboard, featuring most of the IO ports of the original.
The density of utility of the Pi 400 was something I just had to have, even if I didn't have an immediate need for it.

![](assets/img/2023-01-16-keyboard-computer-rpi468/pi400.jpg)
{: centeralign }
The official Raspberry Pi 400 keyboard computer.
{: centeralign }

After a somewhat impulsive purchase from my local Micro Center, I went home with a Pi 400.
It was both incredible and lack luster in the ways I expected: it was a bonafide Raspberry Pi computing platform, and the keyboard was mediocre to type on (mushy and uneven feeling, shallow key travel).
The obvious conclusion that I came to was: why not combine a tactile mechanical keyboard with the main board of the Pi 400?

And so I set to work to combine my trusty (albiet aging) Tada 68 mechanical keyboard with the main board of the Pi 400.
It seemed fitting to combine the names:

Raspberry Pi **400** + Tada **68** = Raspberry Pi **468**.

{% include image-gallery.html folder="/assets/img/2023-01-16-keyboard-computer-rpi468" %}


# Design goals
I drew inspiration from [Pavlo Khmel on YouTube](https://www.youtube.com/watch?v=TTT5TCiPke4&pp=ygUkcmFzcGJlcnJ5IHBpIDQwMCBtZWNoYW5pY2FsIGtleWJvYXJk&ab_channel=PavloKhmel) also upgraded the Pi 400 to be integrated with a mechanical keyboard.
However there were design decisions made in his implementation that I wanted to do differently:
1. The mechanical keyboard is connected to the Pi 400 by running a USB A cable out of the case and then back in. Functionally, there is nothing wrong with this, but aesthetically I could not abide it.
2. When the mechanical keyboard is connected to the Pi 400, it cannot be used by any other host device such as a desktop PC. This is inconvenient if the Pi is meant to be used as a headless server while using the keyboard for a desktop PC.

The goal for my design was to have a seemless experience in operating in either 'keyboard computer' mode, or 'keyboard only' mode (for a host PC).
To achieve this and also eliminate a USB A cable loop outside of the case, I needed to implement a USB switching circuit board.
Another design constraint I imposed was that the form factor of the keyboard be indistinguishable from any similarly sized mechanical keyboards (65% layout).
After some research, I realized this circuit board did not exist in the form factor I needed, and I'd have to design and build a custom one.

For the mechanical design I settled on a two-part case made of 3D printed material.
I decided on mechanical fastening of the two halves for repairability.
I toyed with the idea of a unibody CNC'd aluminum case, but cost and fear of signal degradation took me off this path.
In the second iteration of the case design, I included a CNC'd aluminum heat sink for passive heat dissapation of the Pis CPU and RAM.


# Tools used
Design tools
- For circuit design, I used KiCAD.
- For mechanical design (case), I used SOLIDWORKS.

Services
- For PCB fabrication, I used OSHPark.
- For stencil printing, I used OSHStencils.

Part suppliers
- I ordered my circuit components from DigiKey.
- For mechanical components, I used McMaster.

- For soldering, I used a combination of hot plate and hot air. I also tried a reflow oven but had poor results.


# Challenges
The concept and design of this project were thankfully very straightforward.
Where I struggled most was at the execution level:
- Soldering the USB switcher circuit
- Desoldering and soldering the USB port on the assembled keyboard PCB
- Designing the tighter tolerance features around the Pi 400 PCB: mounting holes, snap clips


# Building one yourself
I honestly hadn't put much thought into what it would take for someone else to replicate this project, but I'll leave some high-level pointers here.

## Buy list

## Assembly

## Testing


# Final result
Overall I'm quite happy with the result.
I can't say I've been daily driving the keyboard in this computer, but that has more to do with the raw compute horse power of the Raspberry Pi 4.

Generally I use it most as a regular keyboard, especially when I travel.
Since my work spaces at the office and at home have their own dedicated keyboard, this one usually stays in my backpack as a backup.
It's comforting to know though that I have all of the low-level hardware capability of a Raspberry Pi with me at all times without any additional weight or backpack space being taken up.
