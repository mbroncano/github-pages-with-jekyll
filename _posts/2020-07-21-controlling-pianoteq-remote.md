---
title: Controlling a headless Raspberry Pi running Pianoteq with an iPad
date: {}
published: true
---
One of the challenges of running Pianoteq in a headless device such as Raspberry Pi is how to remotely navigate through the settings, load presets, and so on. In this article, I'll explain how my current setup works.

### A headless setup
Before I start, these are some of the basic requisites:
* The unit must power on and off unattended
* There should be no more services running at boot than necessary
* The defaults should be appropriate for a regular piano practice session
* Ideally, we should be able to load presets and modify parameters

In this article, we will discuss the last one.

### A bit of history
The MIDI standard is well on its way to becoming 40 years old. A few visionaries decided that it was time to have a single protocol for the dozens of digital synthesizers that appeared during the early 80s.

Originally, MIDI is a relatively slow (around 32kbit/s) serial protocol where a set of messages are sent over 16 logical channels. There are two basic categories of MIDI messages, channel and system messages. System messages are used for different purposes, such as synchronization. They also have provided the means to extend the MIDI protocol over the years with the so-called 'system exclusive messages', which provide vendor-specific features that were not covered in the standard.

The most common type of messages though are the channel ones. They can also be subdivided in further categories, going from the most basic note on/off action to sound bank selection,

### A remote controller
The basic idea is to use some external controller to modify some of the parameters by using MIDI messages. Pianoteq provides the flexiblity to achieve this, allowing channel and program changes to trigger some of the actions in the UI.

Initially I though of using a MIDI pad like the ones that are part of some cheap keyboard you can buy on Amazon, but one the one hand they are more bulky that I would ideally like and on the other hand, I would ideally like more flexiblity and withdraw any cabling requirements.

Using my iPad with some software, that either I would write myself or just buy if available, would be ideal. Using a wireless connection implies a lot of latency, but given the use I have no worries about it. So how do we send MIDI messages over WiFi?

### ~AppleMIDI~ RTP-MIDI to the rescue
Less than a couple of decades ago, some really nice gentlemen working for Apple decided it was time to bring MIDI over to the 21st century. In order to do that, they create [RTP-MIDI](https://en.wikipedia.org/wiki/RTP-MIDI) also known as AppleMIDI (by really nobody) and OSX version 4 included its first incarnation.

Of course, we are not running Pianoteq in our good macOS machines, that would be far too easy. While searching an implementation for Linux, I came up with an apparently mature commercial implementation and another one called [rtpmidid](https://github.com/davidmoreno/rtpmidid) that although considered alpha-stage by its own author, was exactly what I was looking for. The author was kind enough to provide `.deb` packages for ARM as well.

### Setting up `rtpmidid`
### Setting up Pianoteq
### Setting up the iPad
