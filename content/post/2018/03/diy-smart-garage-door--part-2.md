---
title: "DIY Smart Garage Door with HomeKit - Part 2"
date: 2018-03-19T20:09:35+11:00
draft: true
---

The Hardware Build
===

This post will cover the design and build of my hardware solution for the DIY Smart Garage Door Controller, from its humble beginnings on the breadboard (pictured below) to final prototype.  

{{% fluid_img src="/img/garagedoor/breadboard.jpg#center" alt="ESP8266 on breadboard" %}}
  
The initial breadboard circuit was something I threw together to work on the software. It was based on a single momentary switch (button) simulating the door sensor, with an LED simulating the relay. Not a whole lot changed in the design, other than to add a second door sensor. The ability to detect a "fully open" state allowed me to remove the assumption that if the door was not fully closed, it was fully open.
  
One goal of the design was to ensure that it would be simple to remove the NodeMCU board. I wanted to be able to reflash it with new software without the hassle of a full hardware teardown/install each time and without having to lug my laptop out to the garage. The result, while probbaly overkill, was a set of female headers on the main circuit board to house the NodeMCU as well as a set of connectors for the relay and sensors. Now it is simple to replace the either the NodeMCU board or the circuit board (perhaps with a PCB, one day).


Parts List
---

To recreate my build from scratch, you will need the following parts:
  
- ESP8266 (NodeMCU) Development Board
- 1 x 5v Relay board
- 2 x magnetic reed switch
- 1 x perfboard
- 2 x 15-pin single-row female header
- 1 x 3-pin 90-degree male connector
- 1 x 4-pin 90-degree male connector
- 1 x 4-pin female connector
- 2 x 3-pin female connector
- 2 x 10kΩ resistors
- 1 x project box
- various jumper/hookup wires
  
I purchased everything from my local [electronics store](https://www.jaycar.com.au/), except for the NodeMCU board.
My build price was somewhere around AUD$35 (assuming I will use the leftovers for another project), but it could be a lot cheaper sourcing parts from ebay/aliexpress and/or skipping the connectors and soldering the sensor and relay wires directly to the circuit board.

Circuit Board
---
With the goals finalised, I modved on to circuit design and layout. Using [Fritzing](http://fritzing.org/home/), I set out my design for the (top of the) board. Unfortunately the fzz file was corrupted, and I can't be bothered remaking it... but here's a snap I took before I lost the file:

{{% fluid_img src="/img/garagedoor/fritzing.jpg#center" alt="Fritzing circuit board layout" %}}

...and a schematic of the system:

{{% fluid_img src="/img/garagedoor/schematic.jpg#center" alt="system schematic made on easyeda.com" %}}

The build required modification of some parts (cutting the perfboard and converting the 40-pin female header I bought into 2 x 15 pin), followed by soldering the parts into place and creating some solder bridges on the bottom of the board to close a few circuits. The completed board is pictured below, I was pretty happy that I had jumper wires coloured to match the design. Note that I swapped the centre 2 pins on the 4-pin connector to make life a little easier:

{{% fluid_img src="/img/garagedoor/perfboard1.jpg#center" alt="perf board top" %}}
  
{{% fluid_img src="/img/garagedoor/perfboard2.jpg#center" alt="perf board bottom" %}}

Thankfully, I didn't need any rework (this time).
All that's left is to assemble the connections: 2 wires from each reed switch go into the 4-pin female connector, then a 3-wire cable needs to be built to link the relay board to the 3-pin male (take care to connect + to Vcc, - to GND and S to D5). Add 2 wires to C and NO of the relay that connect into the garage door button then find a way to fit it all into the project box.
  
I actually selected a project box that was a little too small, but found another container at home that was up to the task. The final images are of the system packaged into that box, then the complete hardware solution:

{{% fluid_img src="/img/garagedoor/packaged.jpg#center" alt="packaed into project box" %}}
  
{{% fluid_img src="/img/garagedoor/complete.jpg#center" alt="complete solution" %}}

Pictures of the "installation" (a term I used very loosely) are available in my [previous post](../diy-smart-garage-door--part-1/); but essentially you need to have a power source, connections to the button (from the relay) and the reed switches positioned on the door so that they are able to detect fully closed and fully open states. The reed switch connected to pin D1 is for closed state, with D2 used for detecting open.
  
Thanks for reading!