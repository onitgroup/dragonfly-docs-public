# RTP streaming from Raspberry PI

## Introduction

This gist describes the necessary software installation steps for a **Raspberry PI** in order to enable the PI's camera to act as an external camera for the Dragonfly Java application. The Dragonfly Java app and the Raspberry PI need to be on the same network for this or the Raspberry PI must be publicly available. This gist shows, how to make a Raspberry PI streaming RTP over TCP. The resulting feed can then be used as input for the Dragonfly Java app. It cannot be used as input for the [Accuware Dragonfly Demo - Calibration Mode](https://dragonfly-demo.accuware.com) server. For this you need to enable the PI to do RTSP or WebRTC streaming. The appropriate gists are here:

### RTSP streaming from Raspberry PI

<https://gist.github.com/accuware/269a162badfc8e75f444a142b5e0a36a>

### WebRTC streaming from Raspberry PI

<https://gist.github.com/accuware/c67ea205c713fb465adaeb0e506d8f7f>

### WebRTC streaming from Raspberry PI using UV4L directly

<https://gist.github.com/accuware/370c3fdd758b5cb4b41c6aa2acfe9ce6>

### Create and deploy a self-signed certificate for the Raspberry PI

<https://gist.github.com/accuware/b9248e1d2ee8e6e4022557234b7b42a9>

## Prerequisites

- **Raspberry PI Zero W, 2, 3, 3b, 3b+, 4** with a [Raspberry PI-Cam 5 MP](https://www.amazon.de/Raspberry-Pi-Cam-Megapixel-Camera-Board/dp/B00GAIDGQ6) or a [SainSmart Wide Angle Fish-Eye Cam](https://www.amazon.de/SainSmart-Fish-Eye-Camera-Raspberry-Arduino/dp/B00N1YJKFS/ref=asc_df_B00N1YJKFS/?tag=googshopde-21&linkCode=df0&hvadid=310638483583&hvpos=1o3&hvnetw=g&hvrand=12314684975119410116&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9043132&hvtargid=pla-440264587579&psc=1&th=1&psc=1&tag=&ref=&adgrpid=63367893073&hvpone=&hvptwo=&hvadid=310638483583&hvpos=1o3&hvnetw=g&hvrand=12314684975119410116&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9043132&hvtargid=pla-440264587579) (recommended).
- USB cams are not addressed by this gist, even though possible. The configuration of U4VL is slightly different. We don't recommend the use of USB cameras with a Raspberry PI for latency reasons.

- [**Raspbian Stretch Lite**](https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2019-04-09/) or

- [**Raspbian Buster/Buster Lite**](https://www.raspberrypi.org/downloads/raspbian/)

- A recent version of the `GStreamer` on your Mac or Linux PC (1.14 sufficient, 1.16 recommended). Check the GStreamer documentation, how to install it.

>**Please note:** Although technically possible to use any of the a.m. Raspberry PI devices, we recommend to use at least a Raspberry PI 3B+, since it has sufficient computation power and allows you to use 5GHz Wifi, which is mostly not that crowded.

Before first boot and after having flashed the SD card with the image (e.g. using `Etcher`):

- Enable headless SSH by placing an empty `ssh` file into the boot partition `/boot` of the SD
- Enable headless Wifi by placing a `wpa_supplicant.conf` into the boot partition to the boot partition `/boot` of the SD. 5GHz Wifi preferred, if possible.

```ini
country=<your-two-letter-code>
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
       ssid="<your-ssid>"
       psk="<your-password>"
       key_mgmt=WPA-PSK
}
```

On problems with Wifi: <https://www.raspberrypi.org/forums/viewtopic.php?t=191061>

- Find the IP of the PI
- SSH to it

## Installations

After having found out the IP of the IP on your network SSH to the PI:

```bash
ssh pi@<ip-of-pi>
```

Initial password is `raspberry`.

## Update and Upgrade

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Configure the PI once you are at console level

```bash
sudo raspi-config
```

- Change user password. This is from then on your SSH password. **Strongly recommended**.
- Interfacing options/Enable camera

### Reboot the PI

```bash
sudo reboot
```

### SSH to the PI

```bash
ssh pi@<ip-of-pi>
```

Use your changed password now.

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install gstreamer1.0-tools
sudo apt-get install gstreamer1.0-plugins-good
sudo apt-get install gstreamer1.0-plugins-bad
sudo apt-get install gstreamer1.0-plugins-ugly
```

Test your installation. In order to do this run this command on your Pi:

```bash
raspivid -t 0 -w 640 -h 480 -fps 48 -b 2000000 -awb tungsten  -o - | gst-launch-1.0 -v fdsrc ! h264parse ! rtph264pay config-interval=1 pt=96 ! gdppay ! tcpserversink host=0.0.0.0 port=5000
```

If you have installed `GStreamer` via the DMG file installation package, run this command in a terminal on your Mac:

```bash
/Library/Frameworks/GStreamer.framework/Commands/gst-launch-1.0 -v tcpclientsrc host=<your_Pi's_IP> port=5000 ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert  ! osxvideosink sync=false
```

Run this command in a terminal on your Linux PC. The same command works on your Mac, if you have installed GStreamer via `brew`:

```bash
gst-launch-1.0 -v tcpclientsrc host=<your_Pi's_IP> port=5000 ! gdpdepay ! rtph264depay ! avdec_h264 ! videoconvert  ! autovideosink sync=false
```

You should see a video.

Learn about how to use this RTP stream for operation inside the Dragonfly Java app by reading [the online documentation of the Dragonfly Java app](https://gist.github.com/accuware/dee72abcd94bda40ee14cc3d81a2d9cb).

Once the calibration is done, you need to find a way to make the `raspivid` command string launch at startup (e.g. via /etc/rc.local) and make it a service by appending `&`.
