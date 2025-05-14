---
date: '2025-02-12T00:00:00-06:00'
title: 'Meshtastic & NATS'
---

Yesterday I attended a local developer user group meetup. The topic was [Meshtastic](https://meshtastic.org/), a self-described open source, off-grid, decentralized, mesh network built to run on affordable, low-power devices. It uses [LoRa](https://en.wikipedia.org/wiki/LoRa) under the hood.

## Meshtastic Overview

I have never used or heard of Meshtastic or LoRa before. Some interesting bits that stuck stuck out to me:

- Long range: ~1–[205](https://www.reddit.com/r/meshtastic/comments/1fnduwo/mountain_to_mountain_331_km/) miles
- Slow: ~1–10 kbps
- [Open Source](https://github.com/meshtastic)
- It supports **MQTT**

You can use Meshtastic to send group or direct messages. It uses a mesh network (hence the name), meaning that each device will propogate any message it receives, to a certain point. Messages are sent with an integer that is decremented each time it is forwarded. When the integer reaches zero, it is no longer forwarded.

## Meshtastic Device

The presenter gave away a device, of which I was the lucky recipient (thanks Jim)! The device was a [Heltec LoRa 32 Wireless Stick Lite V3](https://meshtastic.org/docs/hardware/devices/heltec-automation/lora32/?heltec=Wireless+Stick+Lite+V3). It is an ESP32 device with no screen. It was already configured with Meshtastic firmware. Here is a [link](https://docs.heltec.org/en/node/esp32/wireless_stick_lite/index.html#) to the Heltec docs.

The first step was to connect the Meshtastic device to my computer. I connected it to my Windows desktop. I would have used my MacBook, but I thought that there were only drivers for Windows. It turns out there are macOS drivers, so I will try that later. Specifically, I needed the [CP210x USB to UART Bridge](https://www.silabs.com/developer-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads) drivers. I don't know what CP10x or UART is. Before connecting the device, I downloaded the zip file and unzipped it. Inside there is a file with instructions and release notes. On my machine it was named `CP210x_Universal_Windows_Driver_ReleaseNotes.txt`.

With the driver ready, I connected the device and ... nothing. Turns out I was using a _charging_ USB-C cable, not a _data_ USB-C cable. I don't know how to tell the difference just by looking at them or if there even is a difference on the outside. Regardless, I found another cable that I knew could send data and tried again. This time Windows recognized the device and prompted me to install drivers. I pointed Windows to the previously unzipped folder and voilà. I verified the device was showing up in `Device Manager > Ports`.

I wasn't able to figure out how to make the device show up in WSL2. There are Linux drivers available, but I didn't spend too much time trying to figure that part out.

From there, I connected to the device using the Meshtastic web client: [https://client.meshtastic.org/](https://client.meshtastic.org/). You'll need to use a Chrome-based browser.

## MQTT with NATS.io

I mentioned above that Meshtastic supports speaking to an MQTT server. This got me thinking immediately about using [NATS](https://nats.io/), which can [act as an MQTT server](https://docs.nats.io/running-a-nats-service/configuration/mqtt). A sample NATS server configuration to enable MQTT looks like this:

```conf
# nats.conf
jetstream: {}
mqtt: {
    listen: "0.0.0.0:1883"
}
```

Note that you must have JetStream enabled, as mentioned in the [NATS MQTT docs](https://docs.nats.io/running-a-nats-service/configuration/mqtt#jetstream-requirements).

I started the NATS server on my Windows desktop. Note that I used `0.0.0.0` to the server is accessible outside of my localhost. `1833` is the default MQTT port.

## Meshtastic → NATS.io

Back in the Meshtatic web client, I configured a few things. First, I connected the device to my local WiFi so it could talk to my NATS server. Then I enabled serial output (I'm not sure if this was totally necessary). Finally, I configured MQTT with the IP address of my Windows machine and the NATS server's MQTT port and a username. It doesn't matter what the username value is in this case, but it does seem to be required by MQTT.

{{< figure
  src="/images/2025-02-12-esp32-wifi.jpg"
  caption="Screenshot of the Meshtastic ESP32 device connected to my WiFi"
>}}

{{< figure
  src="/images/2025-02-12-meshtatic-mqtt.jpg"
  caption="Screenshot of the Meshtastic web client MQTT configuration"
>}}

Finally, I ran `nats sub '>'` on my Windows machine to watch all traffic from the NATS server. Then I sent a message in the Meshtastic web client in one of the channels:

{{< figure
  src="/images/2025-02-12-meshtastic-nats.jpg"
  caption="Success! A Meshtastic message appearing in NATS subscription output"
>}}

<aside>
<span>Meshtastic uses <a href="https://buf.build/meshtastic/protobufs/docs/main:meshtastic#meshtastic.ServiceEnvelope">protobufs</a> and headers, thus the characters before and after the message</span>
</aside>

## What's next?

Next I'd like to try the inverse of what I did: sending a NATS message that shows up in Meshtastic. That will require correctly encoding a protobuf, but I don't imagine that≠i will be too complicated.
