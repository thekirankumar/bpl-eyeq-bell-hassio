# mqttbell
Local MQTT publishing client to run on BPL Eyeq WiFi doorbells. Full credit to @nwaelti for this.

Put the Home assistant (Hassio) MQTT (addon) IP Address into the source and then run automations based on the event from doorbell press. 

An example automation with this : If MQTT event with topic = `cmd/doorbell/dingdong` (provided by this addon) then capture a photo from door bell camera (mjpeg integration has to be done separately), then send a telegram message (telegram integration separate) 

Before compiling: change of the mqtt clientid and home assisant MQTT broker IP address.

# Compilation
Compilation can be achieved on a debian using a cross compile toolchain for mips, with the following command line:

`mips-linux-gnu-gcc -mips32 -muclibc -EL -mabi=32 -static mqttbell.c -o mqttbell`

When I ran this on a debian, it generated a corrupt binary (600KB), so I ran this command on a windows machine which gave the right binary (50KB)

# Installation
Installation on the doorbell is done with the following procedure:

1. Connect to the doorbell using telnet (user: root, password: 123456)
2. Have a TFTP Server running
3. Upload your mqttbell on the doorbell (`tftp -g -r mqttbell <ip_of_tftp_server>`) in the `/system/system/bin` folder
4. Download your `/system/init/ipcam.sh` file to modify it on the debian machine
5. In that file, add two lines: `iptables -t nat -A OUTPUT -p udp -d 112.74.102.136 -j REDIRECT --to-ports 8629` and `/system/system/bin/mqttbell &`
6. Upload the modified file and make it executable (`chmod +x mqttbell`)
7. Start the commands manually to test

Now pushing the ring button on your doorbell should trigger a mqtt message.

Have fun...
