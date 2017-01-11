# mqttbell
local MQTT publishing client to run on [ACTOP/KONG/etc] WiFi doorbells &amp; clones

Before compiling: change of the mqtt clientid and broker IP address.

# Compilation
Compilation can be achieved on a debian using a cross compile toolchain for mips, with the following command line:

mips-linux-gnu-gcc -mips32 -muclibc -EL -mabi=32 -static mqttbell.c -o mqttbell

# Installation
Installation on the doorbell is done with the following procedure:

1. Connect to the doorbell using telnet (user: root, password: 123456)
2. Have a TFTP Server running
3. Upload your mqttbell on the doorbell (tftp -g -r mqttbell ip_of_tftp_server) in the /system/system/bin folder
4. Download your /system/etc/ipcam.sh file to modify it on the debian machine
5. In that file, add two lines: "iptables -t nat -A OUTPUT -p udp -d 112.74.124.169 -j REDIRECT" and "/system/system/bin/mqttbell &"
6. Upload the modified file and make it executable (chmod +x)
7. Start the commands manually to test

Now pushing the ring button on your doorbell should trigger a mqtt message.

Have fun...
