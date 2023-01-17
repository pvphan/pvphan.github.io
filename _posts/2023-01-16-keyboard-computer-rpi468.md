---
layout: post
title: "Raspberry Pi 468: a full computer in a keyboard"
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

After a somewhat impulsive purchase from my local Micro Center, I went home with a Pi 400.
It was both incredible and lack luster in the ways I expected: it was a bonafide Raspberry Pi computing platform, and the keyboard was mediocre to type on (mushy and uneven feeling, shallow key travel).
The obvious conclusion that I came to was: why not combine a tactile mechanical keyboard with the main board of the Pi 400?

And so I set to work to combine my trusty (albiet aging) Tada 68 mechanical keyboard with the main board of the Pi 400.
It seemed fitting to combine the names:

Raspberry Pi 400 + Tada 68 = Raspberry Pi 468.


# Design process
I drew inspiration from others online who had combined a mechanical keyboard with the Pi 400 (shout out to Khmel and X).
However there were compromises made in their implementations that I did not want for my design.

These designs compromised on the following:
1. The mechanical keyboard is connected to the Pi 400 by running a USB A cable out of the case and then back in. Functionally, there is nothing wrong with this, but aesthetically I could not abide it.
2. When the mechanical keyboard is connected to the Pi 400, it cannot be used by any other host device such as a desktop PC. This is inconvenient if the Pi is meant to be used as a headless server while using the keyboard for a desktop PC.

The goal for my design was to have a seemless experience in operating in either 'keyboard computer' mode, or 'keyboard only' mode (for a host PC).
To achieve this and also eliminate a USB A cable loop outside of the case, I needed to implement a USB switching circuit board.
After some research, I realized this circuit board did not exist in the form factor I needed, and I'd have to design and build a custom one.
Another design constraint I imposed was that the form factor of the keyboard be indistinguishable from any similarly sized mechanical keyboards (65% layout).
This size constraint impacted the component selection for the custom USB switching circuit.

# Challenges
The concept and design of this project were thankfully very straightforward.
Where I struggled most was at the execution level:
- Hand soldering the USB switcher circuit
- Desoldering and soldering the USB port on the assembled keyboard PCB
- Designing the tighter tolerance features around the Pi 400 PCB: mounting holes, snap clips

# Final result
Overall I'm quite happy with the result.
I can't say I've been daily driving this keyboard computer, but that has more to do with the raw compute horse power of the Raspberry Pi 4.

Generally I use it most as a regular keyboard, especially when I travel.
Since my work spaces at the office and at home have their own dedicated keyboard, this one usually stays in my backpack as a backup.
It's comforting to know though that I have all of the low-level hardware capability of a Raspberry Pi with me at all times without any additional weight or backpack space being taken up.
