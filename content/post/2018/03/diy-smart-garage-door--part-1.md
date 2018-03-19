---
title: "DIY Smart Garage Door with HomeKit - Part 1"
date: 2018-03-18T20:09:35+11:00
draft: false
---

{{% fluid_img src="/img/garagedoor/complete.jpg#center" alt="Complete DIY Controller Hardware" %}}

While looking for parts of my house that I want to bring into the Smart Home/IOT era, I started searching for DIY "smart" devices to get started on a low budget.  
  
One blog post titled [$10 DIY Wifi Smart Button / Switch](http://www.simpleiothings.com/build-a-10-wifi-smart-button/) caught my attention with the ESP8266 Development Board - this is a very cheap and convenient board that can be powered via USB, flashed using Arduino IDE (along with several other methods) and has built-in WiFi.  
  
I promptly ordered a few dev boards from [AliExpress](https://www.aliexpress.com/item/2015-New-product-Wireless-module-NodeMcu-Lua-Wifi-Nodemcu-WIFI-Network-Development-Board-Based-ESP8266-High/32521100830.html) and started experimenting when they arrived.  
My first "victim" would be the hard-wired button on my Garage Door Opener. By interfacing only with the button, there's no need to modify the opener itself - which is always a win.
  
The initial high-level goal for the project was to control my garage door via our Google Home, but I quickly discovered that Google sends custom command packets from their cloud servers, not directly from the in-home device (to handle requests coming from Google Assistant on any authorised device).  
Apple HomeKit, on the other hand, supports local-only connections. As I wanted to stay on my private network for the first iteration, I decided to implement with HomeKit first and come back to Google later.  

*Enter [HomeControl](https://github.com/brutella/hc)* ...
---

{{% fluid_img src="/img/garagedoor/home.jpg#center" alt="Apple Home UI showing Garage Door" %}}

Matthias Hochgatterer has provided this incredible implementation of HomeKit Accessory Protocol (HAP) in go, so not only did I get to dabble with HomeKit, I got to do it [Gopher Style](https://www.youtube.com/watch?v=uYXX4FCL-uw&t=6) (sorry!).  
  
The software project currently consists of 3 parts:  

- ESP8266 firmware: Handles the sensors and button, with some intelligent open/close control
- Go binary (gdhk): Small server that acts as a HomeKit proxy for the basic HTTP API on the ESP8266
- spk packaging script: I run the proxy on my Synology NAS, this packages it up for simple "installation" and management
  
Now for the code
---
The first iteration of all my code was pushed to [https://github.com/forfuncsake/garagedoor](https://github.com/forfuncsake/garagedoor/tree/73a2f98a15135c128b2301478ea8d917225c66bc).
  
Because I hope to also add Google Assistant support in the future, most of the door control logic is implemented on the ESP8266, with only the HomeKit-specific parts being handled in the proxy.  
  
***ESP8266 Firmware***

The basic HTTP API has 4 endpoints:  

- ` GET /     ` :  Responds with the current state of the door
- `POST /open ` :  Attempt to open the door
- `POST /close` :  Attempt to close the door
- `POST /press` :  Activate the button, regardless of door state

Door state is determined by the state of an "open" sensor (active when the door is fully open) and a "closed" sensor (active when the door is fully closed). If the door is neither fully open or closed, it must be partly open. My door takes 15-16 seconds to transition between states, so I added a 16 second timer to track the transitional states of "opening" and "closing".  
  
When sending an `open` or `close` command, the API will only activate the button if the door is known to be in the expected "starting state". If requested while the door is known to be in transition, an `HTTP 400` is returned (and obviously, the button is not activated).  
  
Any time a change of state is detected in either of the sensors, a refresh request it sent to the HomeKit proxy. The proxy then captures the new state and - if required - sends an `EVENT` to connected HomeKit clients. This means I get notifications on my iPhone when the door opens and closes, even via the existing button or remotes.  
  
***HomeKit Proxy in Go***

The implementation of the HomeKit proxy is very simple, thanks to the `hc` project.  
I simply needed to create a new "Accessory" type that has a garage door opener and also a switch; and implement a few functions to proxy requests to the API on the ESP8266. The *opener* interfaces to the API to get the current door status and to send "open" and "close" requests. The *switch* is connected to the "/press" endpoint to enable explicit button presses (to stop the door part way, or resume if it was previously stopped).

***Packaging the proxy for Synology***

Having worked extensively with SynoCommunity's spksrc in the past, I knew the simplest way to get a service running on my Synology (and persisting over reboots) was to package it up as in SPK.  
Thankfully, the process is quite straightforward. I created the requisite Makefile and scripts, put my cross-compiled binary in the right place, ran `make` and ended up with a neatly packaged spk for manual installation via the Synology DSM Web UI.  

For the time being, the required files are commited to my own git project, but they can be used as follows:  

```
> git clone https://github.com/forfuncsake/garagedoor
> cd garagedoor/cmd/gdhk
> GOOS=linux GOARCH=arm GOARM=5 go build #(adjust env vars for your Synology architecture)
> cd ../../..
> git clone https://github.com/synocommunity/spksrc
> cp -r garagedoor/spksrc/spk/garagedoor spksrc/spk/
> cd spksrc/spk/garagedoor
> make
```

What Next?
---

I plan to refine all pieces of the software to make the solution more configurable and easier to share. See the project [Issues](https://github.com/forfuncsake/garagedoor/issues) list for visibility into known issues and roadmap items.
  
---

Stay tuned for Part 2, in which I will detail the hardware design and build, which has cost me somewhere around AUD$35.

Here it is "installed" (literally hanging from a loose screw and blu-tac while I test). It's actually been working really well:

{{% fluid_img src="/img/garagedoor/installed.jpg#center" alt="Project installed for testing on Garage Door" %}}