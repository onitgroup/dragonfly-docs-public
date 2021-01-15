# RTSP streaming from Raspberry PI

## Introduction

This gist describes the necessary software installation steps for a **Raspberry PI** in order to enable the PI's camera to act as an external camera for the Dragonfly Java application. This gist shows, how to make a Raspberry PI an RTSP streaming server. The resulting feed can then be used as input for the Dragonfly Java app or the [Accuware Dragonfly Demo - Calibration Mode](https://dragonfly-demo.accuware.com) server. The RTSP server on the Raspberry PI must be made publicly available, if calibration is a requirement.

## Prerequisites

- **Raspberry PI Zero W, 2, 3, 3b, 3b+, 4** with a [Raspberry PI-Cam 5 MP](https://www.amazon.de/Raspberry-Pi-Cam-Megapixel-Camera-Board/dp/B00GAIDGQ6) or a [SainSmart Wide Angle Fish-Eye Cam](https://www.amazon.de/SainSmart-Fish-Eye-Camera-Raspberry-Arduino/dp/B00N1YJKFS/ref=asc_df_B00N1YJKFS/?tag=googshopde-21&linkCode=df0&hvadid=310638483583&hvpos=1o3&hvnetw=g&hvrand=12314684975119410116&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9043132&hvtargid=pla-440264587579&psc=1&th=1&psc=1&tag=&ref=&adgrpid=63367893073&hvpone=&hvptwo=&hvadid=310638483583&hvpos=1o3&hvnetw=g&hvrand=12314684975119410116&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9043132&hvtargid=pla-440264587579) (recommended).
- USB cams are not addressed by this gist, even though possible. The configuration of U4VL is slightly different. We don't recommend the use of USB cameras with a Raspberry PI for latency reasons.

- [**Raspbian Stretch Lite**](https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2019-04-09/) or

- [**Raspbian Buster/Buster Lite**](https://www.raspberrypi.org/downloads/raspbian/)

- **Ubiquity Robotics Raspberry PI image**. This gist refers to the 2019-06-19 image [available here](https://downloads.ubiquityrobotics.com/pi.html). Basically it is an Ubuntu 16.04 Mate derivate.

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
sudo apt-get install -y gstreamer1.0-tools gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav
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

You should see a video. This is already H.264, but not RTSP.

So we are going to enable this now. Terminate the `raspivid` server on your PI by typing CTRL-C.

```bash
sudo apt-get install libglib2.0-dev
sudo apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
```

Download `gst-rtsp-server` from <https://gstreamer.freedesktop.org/src/>. Should match the installed gstreamer version. This can be checked by

```bash
dpkg -l | grep gstreamer
```

At the time of writing this is `1.14.4` for Buster lite. Please edit the commands below for other versions (i.e. replace all occurrences of `1.14.4` with the code of the version you are using).

```bash
wget https://gstreamer.freedesktop.org/src/gst-rtsp-server/gst-rtsp-server-1.14.4.tar.xz
tar -xf gst-rtsp-server-1.14.4.tar.xz
cd gst-rtsp-server-1.14.4
./configure
make
sudo make install
```

Test your setup:

```bash
cd examples
./test-launch --gst-debug=3 '( videotestsrc !  x264enc ! rtph264pay name=pay0 pt=96 )'
```

If you have installed `GStreamer` via the DMG file installation package, run this command in a terminal on your Mac:

```bash
/Library/Frameworks/GStreamer.framework/Commands/gst-launch-1.0 -v rtspsrc location=rtsp://<your_Pi's_IP:8554/test latency=0 buffer-mode=auto ! decodebin ! videoconvert ! osxvideosink sync=false
```

Run this command in a terminal on your Linux PC. The same command works on your Mac, if you have installed GStreamer via `brew`:

```bash
gst-launch-1.0 -v rtspsrc location=rtsp://<your_Pi's_IP>:8554/test latency=0 buffer-mode=auto ! decodebin ! videoconvert ! autovideosink sync=false
```

You should see a test video.

Terminate the RTSP server on the PI with CTRL-C.

Now we want to see the camera. For this we use the nice `raspivid` wrapper `gst-rpicamsrc`.

```bash
cd ~
sudo apt-get install git
git clone https://github.com/thaytan/gst-rpicamsrc.git
```

### For Raspbian OS

Proceed with

```bash
cd gst-rpicamsrc
./autogen.sh
make
sudo make install
```

### For Ubiquity Robotics OS

As the time of this writing the Raspbian build procedure does not work on Ubiquity. While compilation and linking is going fine, there is a runtime issue:

`mmal: mmal_component_create_core: could not find component 'vc.ril.camera'`

The problem has been discussed and resolved [here](https://github.com/thaytan/gst-rpicamsrc/issues/92).

The workaround is to use `meson` and `ninja` in order to build the camera driver. Since the default version of `meson` is not sufficient for the build, you need to install `meson` via the Python3 installer in order to get a more recent `meson` version.

Proceed with:

```bash
cd gst-rpicamsrc
pip3 install --user meson
mkdir build
~/.local/bin/meson --prefix=/usr build
ninja -C build -v
cd build
sudo ninja install
```

### Finally for both OS

Check, if `gst-rpicamsrc` is installed

```bash
gst-inspect-1.0 | grep rpicamsrc
```

Now for the final test:

```bash
cd ../gst-rtsp-server-1.14.4/examples
./test-launch --gst-debug=3 "( rpicamsrc bitrate=8000000 awb-mode=tungsten preview=false ! video/x-h264, width=640, height=480, framerate=30/1 ! h264parse ! rtph264pay name=pay0 pt=96 )"
```

If it runs, remove `--gst-debug=3` and let it run as a deamon by appending `&` to the command line above.

If you have installed `GStreamer` via the DMG file installation package, run this command in a terminal on your Mac:

```bash
/Library/Frameworks/GStreamer.framework/Commands/gst-launch-1.0 -v rtspsrc location=rtsp://<your_Pi's_IP>:8554/test latency=0 buffer-mode=auto ! decodebin ! videoconvert ! osxvideosink sync=false
```

Run this command in a terminal on your Linux PC. The same command works on your Mac, if you have installed GStreamer via `brew`:

```bash
gst-launch-1.0 -v rtspsrc location=rtsp://<your_Pi's_IP>:8554/test latency=0 buffer-mode=auto ! decodebin ! videoconvert ! autovideosink sync=false
```

For more insight into `gst-rpicamsrc` and possible other parameters as already used above:

<https://sparkyflight.wordpress.com/tag/gst-rpicamsrc/>

Learn about how to use this RTSP streamer for operation and calibration from inside the Dragonfly Java app by reading [the online documentation of the Dragonfly Java app](https://gist.github.com/accuware/dee72abcd94bda40ee14cc3d81a2d9cb). You need to make the RTSP server run even if you have closed the shell. This is out of the scope of this gist.

For the normal Dragonfly Java app operation it is sufficient to use the PI just as RTP streamer. For this if you have finally finished the calibration, you need to find a way to make the `raspivid` command string launch at startup (e.g. via /etc/rc.local) and make it a service by appending `&`.
