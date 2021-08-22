---
title: "Owning My Own Powerpal Data"
date: 2021-08-22T17:30:00+10:00
draft: false
tags: ["IoT", "Powerpal", "Energy Monitoring", "BLE", "golang", "mitm", "mitmproxy"]
---

If you haven't heard of Powerpal, it's a little Bluetooth Low-Energy (BLE) device available in Australia that attaches to your residential electricity meter to collect and display realtime and historic energy usage data via a mobile app.
This alone is pretty awesome, but if you're in the state of Victoria, you can also get one for free under the Victorian Government Energy Upgrades scheme.

{{% fluid_img src="https://storage.googleapis.com/powerpal/images/product-hero.jpg?v=4#center" alt="Powerpal device with iPhone running the Powerpal app" %}}

... and if you claim yours via my referral link, I get $5: https://www.powerpal.net/invite/4L997H9C


Being able to see realtime usage data is great for finding power hogs in the home, but you can only view your data in the app or via manually exported CSV files of up to 90 days each, and the mobile device needs to be in BLE range to see realtime usage.

Here's the good news: When you pair additional mobile devices to your Powerpal, only one device needs to be in BLE range; it will upload the data to Powerpal servers periodically and your other devices can poll the server for updated (~15min delayed chunks) meter readings. But we're still restricted to only viewing data in the app.

I run HomeAssistant and would love to be able to add my power usage to a dashboard, and maybe even use it to trigger notifications or events. In researching if this would be possible, I came across a [whirlpool thread](https://forums.whirlpool.net.au/archive/322nx083) where plenty of people have similar ideas; and that's where I learned about the aforementioned remote viewing capabilities (on a second mobile device) and that there is a [partner API](https://readings.powerpal.net/documentation) available.

I quickly set up the app on our iPad, paired to the Powerpal, switched off bluetooth, reconnected my phone to the Powerpal and checked my PiHole dashboard. It showed that the iPad was looking up `readings.powerpal.net` (the same server hosting the partner API). Was it actually just using the same API? (I mean, why would you build two APIs?)

## Enter mitmproxy

This is not a guide on setting up mitmproxy, so at a very high level, my next steps were:

  - `brew install mitmproxy`
  - run `mitmweb`
  - [set up iOS](https://www.howtogeek.com/293676/how-to-configure-a-proxy-server-on-an-iphone-or-ipad/) to use mitmweb (`server: <macbook-ip> port: 8080`) as the HTTP proxy
  - [set up](https://docs.mitmproxy.org/stable/concepts-certificates/) the root cert trust on the iPad
  - re-launch the Powerpal app
  - capture and inspect HTTP flows

Inspecting the flows confirmed that the app simply makes calls to the documented `readings` API. The API token used in the HTTP Authorization header was the same on both of my mobile devices, suggesting that it's stored on (or derived from) the Powerpal device, because my phone and iPad never negotiated with each other and there was no login phase in the app setup.

At this point, I had all of the pieces needed to make API queries outside of the app... so I wrote a go utility!
My repo can be found at https://github.com/forfuncsake/powerpal. It supports querying the Powerpal API and exporting data to either CSV or InfluxDB. To run the utility, you will need to extract your own authorization token with mitmproxy or similar, after which the process to get the last 2 hours of data in CSV format looks a little something like:


```
ffs@shadow powerpal % git clone https://github.com/forfuncsake/powerpal
ffs@shadow powerpal % cd powerpal
ffs@shadow powerpal % read -s POWERPAL_DEVICE_SERIAL
# enter your powerpal serial number, hit Enter
ffs@shadow powerpal % read -s POWERPAL_API_TOKEN
# enter your extracted powerpal api token, hit Enter
ffs@shadow powerpal % export POWERPAL_DEVICE_SERIAL
ffs@shadow powerpal % export POWERPAL_API_TOKEN
ffs@shadow powerpal % go run ./examples/powerpal2csv -t 2h
2021/08/17 22:41:03 Retrieved 120 new readings, saving to powerpal_export_1629204063.csv
ffs@shadow powerpal %
```

With InfluxDB, we can load all available historic data on the first run, then just pull deltas on subsequent runs. To use with default values, set up an InfluxDB instance with an org value of "home", and a bucket named "data". You will also need to obtain an InfluxDB API key from your server, then:

```
ffs@shadow powerpal % git clone https://github.com/forfuncsake/powerpal
ffs@shadow powerpal % cd powerpal
ffs@shadow powerpal % read -s POWERPAL_DEVICE_SERIAL
# enter your powerpal serial number, hit Enter
ffs@shadow powerpal % read -s POWERPAL_API_TOKEN
# enter your extracted powerpal api token, hit Enter
ffs@shadow powerpal % read -s INFLUX_API_TOKEN
# enter your influxDB api token, hit Enter
ffs@shadow powerpal % export POWERPAL_DEVICE_SERIAL
ffs@shadow powerpal % export POWERPAL_API_TOKEN
ffs@shadow powerpal % export INFLUX_API_TOKEN
ffs@shadow powerpal % go run  ./examples/powerpal2influxdb -s http://<your influx instance address>:8086 -o home -b data -m powerpal
2021/08/22 13:42:46 No last record found, fetching all data after 1970-01-01 10:00:00 +1000 AEST...
2021/08/22 13:42:46 Retrieved 27204 new readings
writing to influxdb
ffs@shadow powerpal % sleep 300
ffs@shadow powerpal % go run  ./examples/powerpal2influxdb -s http://<your influx instance address>:8086 -o home -b data -m powerpal
2021/08/22 13:48:03 Retrieved 5 new readings
writing to influxdb
ffs@shadow powerpal %
```

All of the raw data points (meter pulses, watt hours, cost, etc, per minute) are now in the InfluxDB instance, and can be queried to present in any way you like. Here's a screenshot of the daily totals from the Powerpal app against the same data represented in InfluxDB:

{{< fluid_imgs
  "pure-u-1-2|/img/powerpal/pp-august-daily.png|powerpal app graph of daily totals for august 2021"
  "pure-u-1-2|/img/powerpal/influx-august-daily.png|influxdb graph of daily totals for august 2021"
>}}

My next goal is to use the data in HomeAssistant to alert me if the per-minute usage is "high" after 10pm (suggesting the heating might have been left on), or if weekly total consumption is looking excessive. I hope others can make use of the tools to extract, monitor and archive their own Powerpal data too!
