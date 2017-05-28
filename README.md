## SonOTA
### Flashing Itead Sonoff devices via original OTA mechanism

This program is based on research documented here:

- https://blog.nanl.de/2017/05/sonota-flashing-itead-sonoff-devices-via-original-ota-mechanism/
- https://github.com/arendst/Sonoff-Tasmota/issues/476

The script is fairly well documented (I think) - apart from that above stated links should provide some context of what's been done how and why. Feel free to open issue tickets or contact me directly: sonota@nanl.de

#### DNS redirect

The program only works if you made sure the hostname 'eu-disp.coolkit.cc' is being resolved to an IP address assigned to the machine this script is running on.

On a WiFi router running OpenWrt this can be easily done by adding the following line to `/etc/hosts` and restarting `dnsmasq`:

`10.23.42.5	eu-disp.coolkit.cc`

Adjust IP address and make sure variable `serving_host` in `sonota.py` is set to the same.

#### Creation of SSL certificates

`openssl req -x509 -nodes -newkey rsa:2048 -keyout ${SONOTA}/ssl/server.key -out ${SONOTA}/ssl/server.crt -days 42235`

#### Building an user2 image with Arduino

```
--- a/~/.arduino15/packages/esp8266/hardware/esp8266/2.3.0/tools/sdk/ld/eagle.flash.1m64.ld
+++ b/~/.arduino15/packages/esp8266/hardware/esp8266/2.3.0/tools/sdk/ld/eagle.flash.1m64.ld
@@ -7,7 +7,7 @@ MEMORY
   dport0_0_seg :                        org = 0x3FF00000, len = 0x10
   dram0_0_seg :                         org = 0x3FFE8000, len = 0x14000
   iram1_0_seg :                         org = 0x40100000, len = 0x8000
-  irom0_0_seg :                         org = 0x40201010, len = 0xf9ff0
+  irom0_0_seg :                         org = 0x40281010, len = 0xf9ff0
 }
 
 PROVIDE ( _SPIFFS_start = 0x402FB000 );
 ```

#### Converting Arduino images to v2 images

`esptool.py elf2image --version 2 image_user1.ino.elf`
`esptool.py elf2image --version 2 image_user2.ino.elf`

`mv image_user1-0x01000.bin image_user2-0x81000.bin ${SONOTA}/static/`

#### Run

The script needs to be run as root, as it binds to port 443 to which the Sonoff device initially sets up a HTTPS POST request.

`sudo python sonota.py`


