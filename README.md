# Introduction
Ever wondered how awesome it would be to not have your wifi doorbell's app running on your phone in the background consuming your battery and still get notified.
Wouldn't it be cool if you can get a telegram message on your phone along with the photo of the person at your door?
Or how about having an option to toggle the door lock from within telegram, without opening those shitty chinese apps which hardly work?

All this is possible by using this addon, along with a bunch of other addons. Keep reading ahead.

# What is this ?
This script enables a local MQTT publishing client to run on BPL Eyeq WiFi doorbells. Full credit to @nwaelti for figuring this out.
All you have to do is to put the Home assistant (Hassio) MQTT (addon) IP Address into the source, compile it and push it run on the doorbell, and then run automation based on the event from doorbell press. 

An example automation with this : If MQTT event with topic = `cmd/doorbell/dingdong` (provided by this addon) then capture a photo from door bell camera (mjpeg integration has to be done separately), then send a telegram message (telegram integration separate) 

Before compiling: change of the mqtt clientid and home assisant MQTT broker IP address.

# Pre-requisites
- Make sure your doorbell is manufactured by BPL and you use https://play.google.com/store/apps/details?id=com.bpliq.smartbell to monitor it.
- Make sure you know the IP address of your doorbell and are able to reach this IP via `telnet <ip_of_the_bell>`
- Make sure your home assistant is setup and has a static IP address.
- Make sure both the doorbell and home assistant are able to see each other (as long as they are connected to the same router, this should be fine)
- Make sure you have MQTT server setup on some machine. Simplest setup usually is to install MQTT broker on your home assistant via https://github.com/home-assistant/hassio-addons/blob/master/mosquitto/DOCS.md

# Compilation
Compilation can be achieved on a debian using a cross compile toolchain for mips, with the following command line:

`mips-linux-gnu-gcc -mips32 -muclibc -EL -mabi=32 -static mqttbell.c -o mqttbell`

When I ran this on a debian, it generated a corrupt binary (600KB), so I ran this command on a windows machine which gave the right binary (50KB)
For windows, I installed "Sourcery CodeBench Lite for MIPS GNU/Linux" and ran the same command as above.

# Installation
Installation on the doorbell is done with the following procedure:

1. Connect to the doorbell using telnet (user: root, password: 123456)
2. Have a TFTP Server running
3. Upload your mqttbell on the doorbell (`tftp -g -r mqttbell <ip_of_tftp_server>`) in the `/system/system/bin` folder
4. Download your `/system/init/ipcam.sh` file to modify it on the debian machine
5. In that file, add two lines: `iptables -t nat -A OUTPUT -p udp -d 112.74.102.136 -j REDIRECT --to-ports 8629` and `/system/system/bin/mqttbell &`
6. Upload the modified file and make it executable (`chmod +x mqttbell`)
7. Start the commands manually to test
8. Uninstall the shitty BPL eyeq app from your phone. This addon will always be online on the doorbell unlike their app which crashes once in a while. 

Now pushing the ring button on your doorbell should trigger a mqtt message.

Have fun...

# How does it work
- `ipcam.sh` is the script which launches during bootup of the bell. Hence the two commands `iptables` and `mqttbell` are added to it.
- when a doorbell is pressed, it sends a udp packet to `112.74.102.136` (figured out via wireshark)
- running the `iptables` command redirects a copy of all the udp packets with destination = `112.74.102.136` (chinese server) to the local port `8629`
- mqttbell binary running in the background intercepts all packets coming from the above step (localhost:8629) and makes a MQTT connection and publishes these packets on the MQTT topic `cmd/doobell/dingdong`
- home assistant or any other system subscribed to the above MQTT topic can receive this and use it for automations (like sending a telegram notification or playing a sound).

# Warning
The doorbell keeps sending a hearbeat packet (every minute) to the same server. This packet will also be forwarded to your MQTT. You will have to ignore this message by having a condition on the length. The actual packet sent when the doorbell is pressed will have a larger length than the heartbeat packet.

# Future
The next set of steps would be to figure out a way to use an addon like facebox to recognize the face and use google home TTS engine to to announce who's at your door.
