# Dragonfly Java Dashboard and REST API

## Version 1.7 feature table

| Feature                                              | Supported |
|------------------------------------------------------|-----------|
| Java standalone app, available as obfuscated JAR     | ✔         |
| USB, internal or external camera support             | ✔         |
| Web server and REST interface for control and status | ✔         |
| SSL support                                          | ✔         |
| GZIP support                                         | ✔         |
| Local map save and load                              | ✔         |
| Local map update and delete                          | ✔         |
| Configuration read/write available via REST          | ✔         |
| Map synchronization                                  | ✔         |
| Support for camera capture, native, mono and stereo  | ✔         |
| Offline map display                                  | ✔         |
| Position upload, history, custom server              | ✔         |
| Position logging to CSV                              | ✔         |
| Local video recording                                | ✔         |
| Support for virtual markers                          | ✔         |
| Support for plain X, Y and Z display                 | ✔         |
| Support for pitch, yaw, roll and rotation matrix     | ✔         |
| Support for version update check                     | ✔         |
| Support for metric coordinates                       | ✔         |
| Map point API                                        | ✔         |
| WebRTC support                                       | ✔         |
| REST API authentication support                      | ✔         |
| Camera calibration support                           | ✔         |
| Mono depth prediction support (optional on install)  | ✔         |

## Functions of the Java app

The Java app, controlled and monitored by this dashboard, makes use of an internal or external camera and feeds the captured images to native code, which finally enables it to determine the position of the device and to 3D-map the environment.

Because **3D-Mapping** is one of the central tasks of the Java app and the item "map" or "mapping" is all the way used in this documentation, here a short primer regarding **"What is a map?"** in the context of Dragonfly:

A map is the virtual representation of what has been already seen by Dragonfly. It corresponds to a file, that stores all the information describing a visual environment and enables the tracking of the positions of a camera fitted device in this environment later on, hence "no map, no positioning".

A map contains:

- A store of all the markers that have previously been detected with their position, printed markers as well as virtual markers (reference points, see below)
- All the 3D points corresponding to the 3D reconstruction of the environment
- All the features that have been detected, together with a visual representation of these features

Even though the dashboard is fully capable of loading, storing and manipulating those maps from a local, non-Internet connected environment, there is a special branch of the dashboard responsible for **synchronizing** maps with a central repository hosted at Accuware. This enables collaborative work between different parties on the maps. Please refer to section [Map Synchronization](#map-synchronization) for details on how the synchronization process is working and what basic principles apply.

The Java app exposes a Web server interface on a configurable port and a REST API for remote control. It also hosts a dashboard.

The Java app also provides means to calibrate a camera. Please refer to section [Camera Calibration](#camera-calibration) for details.

There is an optional means to improve the tracking accuracy since version 1.6. Please refer to section [Mono depth prediction support](#mono-depth-prediction-support) for details.

## Markers

`Markers` are an important part of the positioning process. They help in determining the relation of the internally used virtual coordinates to the coordinate system of the "real" world.

Markers can be made known to the system in two different ways:

- In the form of `printed visual markers (aka real markers)`, which are permanently installed at previously defined coordinates and recognized at runtime.
- In the form of `manually fixed reference points (aka virtual markers)`, which are transmitted via this dashboard to the Dragonfly Java App exactly at the time when the device is at this point.

For the first method the [**Accuware Dashboard**](https://dashboard2.accuware.com/dragonfly/marking) provides a page, which helps you to create, configure and print the markers.

In case, placing printed visual markers is not an option, you can alternatively pin-point the current location of the device live on the floor plan. Check out the `Quick help` given on the `Map calibration` tab of this dashboard for details.

In any way, these two methods have to be applied only once in the entire mapping process for a level.

## WebRTC

The Java app supports **WebRTC** in various variants as a means to feed external video. Since it can be a very challenging task to setup all required software and configuration, we don't promote this way actively. Within this documentation you will find just scattered hints about the configurations within the Dragonfly Java app. For deeper insight please refer to the [Backup](#backup) section.

## How the app is deployed

The app comes packed in a ZIP file. Unpack into a directory of your choice and launch the app by simply typing

```bash
java -jar dfja-1.7-dist.jar
```

in a terminal window.

The app requires Java 11 JDK minimum. Check your environment with `java -version`. If the given JDK version is `11` or higher, you are OK to go.

There is also an experimental Docker deployment.

## Functions of the dashboard

The dashboard you are currently using is a ReactJS static web app hosted by the Java app. It is providing currently four tabs

- Main
- Maps
- Configuration
- Documentation

The app makes use of the [REST API](#rest-api), also hosted by the Java app. It generally provides all functionality available via REST plus means to fetch and use floor plans.

Even though web based, the app itself does not necessarily need to have Internet connectivity. This applies with the exception of the initialization of the floor plan display as described below.

The `Main` tab shows a tabular overview of status parameters obtained using the [Get status](#get-status) API. Additionally it allows to start/stop a mapping and/or positioning. It displays the position on a floor plan view. If the site has floor plans, it downloads, caches and displays it, together with available geo fence information, if appropriate location and level information is provided by the core. It uses the site username and password information provided in configuration. Whenever Internet is available, the app checks for fresh floor plan and geo fences in the Accuware cloud. Generally an Internet connection is at least required once in order to download and cache floor plans and geo fences. Additionally means to pin-point real-time positions on floor plans are provided.

The `Map` tab allows to load, save, reset, update, delete local maps and sync local maps with the remote server. Maps are the results of the 3D mapping of the visual environment.

The `Configuration` tab provides means to configure the Java app. Other utility functions are offered.

The `Calibration` tab allows you to calibrate a camera defined by the `CAM_SOURCE` configuration. See also section [Camera Calibration](#camera-calibration).

The `Documentation` tab contains this documentation.

If default parameters are applied, the dashboard is available at `http://localhost:5000`. No authentication required. The webserver supports SSL and HTTP BASIC AUTHENTICATION (`WEBSERVER_AUTHENTICATION_REQUIRED`). It enforces authentication for all REST API requests coming from other servers than localhost.

In case you have a valid certificate, please provide the path to `WEBSERVER_KEYSTORE_FILE` and `WEBSERVER_KEYSTORE_PASSWORD` (best encrypted, see ["Encrypt string"](#encrypt-string)) in `./data/dragonfly2.properties` and set `WEBSERVER_SSL_SUPPORTED` to `true`.

>**Note:** The use of self-signed server certificates requires each `curl` REST API call to be decorated with the extra parameter `-k` in order to make `curl` accepting the certificate. Ex: `curl https://localhost:5000/api/v1/utils/config -k`. You will additionally be requested to trust the self signed certificate, if you open the dashboard the first time in your browser. If `WEBSERVER_AUTHENTICATION_REQUIRED` is `true`, please provide `SITE_USERNAME:SITE_PASSWORD` along with curl parameter `-u`, if you call the API from another server than localhost. `SITE_PASSWORD` has to be provided in its un-encrypted form.

## Start of the Java app

In a terminal window:

```bash
java -jar dfja-1.7-dist.jar
```

The app is logging to console and to a daily rolling log file in `./data/logs`.

## Configuration

Configuration can be provided by either a Java property file (`./data/config/dragonfly2.properties`) or the dashboard, page "Configuration".

The configuration file does not necessarily have to be available at very first startup. Mandatory parameters will be determined in dialog with user and file will be created.

There are currently only two mandatory parameters, listed first below.

| Configuration Item                  | Type    | Meaning                                                                                                                                                                                                                          | Default                             |
|-------------------------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------|
| SITE\_ID                            | String  | **MANDATORY** Accuware Site ID                                                                                                                                                                                                   | n.a.                                |
| SITE\_USERNAME                      | String  | **MANDATORY** Accuware Site Username                                                                                                                                                                                             | n.a.                                |
| SITE\_PASSWORD                      | String  | Accuware Site Password (un-encrypted or encrypted), see ["Encrypt string"](#encrypt-string)                                                                                                                                      | n.a.                                |
| WEBSERVER\_PORT                     | Integer | Local web server port                                                                                                                                                                                                            | 5000                                |
| WEBSERVER\_SSL\_SUPPORTED           | Boolean | Local web server SSL supported (requires WEBSERVER\_KEYSTORE\_FILE and WEBSERVER\_KEYSTORE\_PASSWORD)                                                                                                                            | false                               |
| WEBSERVER\_AUTHENTICATION\_REQUIRED | Boolean | Controls, if REST API requests from other than localhost need to use HTTP BASIC AUTHENTICATION using `SITE_USERNAME` and `SITE_PASSWORD` (un-encrypted). It is recommended to also enable SSL support, if this setting is `true` | false                               |
| WEBSERVER\_KEYSTORE\_FILE           | String  | Path to keystore.jks file                                                                                                                                                                                                        | n.a.                                |
| WEBSERVER\_KEYSTORE\_PASSWORD       | String  | Password for keystore.jks file (un-encrypted or encrypted), see ["Encrypt string"](#encrypt-string)                                                                                                                              | n.a.                                |
| WEBSERVER\_WEBSOCKET\_ENABLED       | Boolean | Enables use of status pushes via websocket                                                                                                                                                                                       | true                                |
| LOG\_LEVEL                          | Integer | Controls granularity of logging. 0: off, 1: fatal, 2: error, 3: warn, 4: info, 5: debug, 6: trace, 7: all                                                                                                                        | 7                                   |
| JMX\_REMOTE\_MONITORING\_ENABLED    | Boolean | Enables monitoring via JConsole. Authorization/Authentication taken from files `./data/config/jmx_access` and `./data/config/jmx_password`                                                                                       | false                               |
| JMX\_RMI\_REGISTRY\_PORT            | Integer | JMX registry port, required to form the JMX Service URL                                                                                                                                                                          | 33985                               |
| JMX\_RMI\_SERVER\_PORT              | Integer | JMX server port, required to form the JMX Service URL                                                                                                                                                                            | 33986                               |
| AUDIBLE\_STATUS\_ENABLED            | Boolean | Gives audible feedback on status changes and marker detection                                                                                                                                                                    | true                                |
| CAM\_MODE                           | String  | Controls, in which mode the camera(s) should be operated, `mono`, `stereo` or `rbg-d` (the latter is to activate mono depth prediction support for monocular cams, if support has been installed)                                | "mono"                              |
| CAM\_SOURCE                         | String  | GStreamer pipe definition, FFMPEG parameter, index or list of comma separated indexes. Default -1 (default camera) (see ["CAM_SOURCE explained"](#cam_source-explained)).                                                        | "-1"                                |
| CAM\_PREVIEW\_OPTION                | Integer | Visible feedback of the image processing. 0: no preview, 1: preview enabled (websockets must be enabled)                                                                                                                         | 1                                   |
| CAM\_FOV                            | Number  | Programmatically rectify the image and set a custom field-of-view. Value should be above 0 and below the actual FOV of the camera (max 180). Default 0 (not applied)                                                             | 0                                   |
| CAM\_IMAGE\_WIDTH                   | Integer | Camera capture and preview image width                                                                                                                                                                                           | 640                                 |
| CAM\_IMAGE\_HEIGHT                  | Integer | Camera capture and preview image height                                                                                                                                                                                          | 480                                 |
| CAM\_CALIBRATION\_FILE              | String  | Filename of the JSON camera calibration params file created by the camera calibration process, located in ./data/config                                                                                                          | ""                                  |
| MAP\_CALIBRATION\_METHOD            | Integer | Number of markers required to be detected, either 0 or 3. If you are using printed or virtual markers, the number has to be 3. If you configuring 0, positions are displayed in `Direct view` only, w/o any geo-reference.       | 3                                   |
| POSITION\_UPLOAD\_INTERVAL          | Integer | Number of milliseconds between two consecutive uploads of positions to the Accuware Dashboard (for tracking the `deviceId`)                                                                                                      | 0                                   |
| POSITION\_UPLOAD\_STORE             | Boolean | Controls, if uploaded positions will be stored in the Accuware Cloud                                                                                                                                                             | false                               |
| POSITION\_LOG\_ENABLED              | Boolean | Controls, if positions should be saved locally (daily rolling CSV log `./data/logs/positionlog.csv`, see CSV logging)                                                                                                            | false                               |
| MAP\_FUSION\_ENABLED                | Boolean | Controls, if map fusion should be enabled or not                                                                                                                                                                                 | false                               |
| VIDEO\_RECORDING\_ENABLED           | Boolean | Controls, if the video taken from the camera(s) should be stored locally, folder `./data/recorded`, grouped by siteId                                                                                                            | false                               |
| VPS\_SERVER\_URL                    | String  | Allows you to overwrite the VPS server URL (Note: provide the entire initial path as shown in default)                                                                                                                           | <https://cvnav.accuware.com/api/v1> |
| PATH\_TO\_CHROMIUM                  | String  | Path to an installed Chromium or Chrome browser instance. If empty, `WebRTC` functionality will be performed using `GStreamer`, if possible                                                                                      | empty                               |
| PATH\_TO\_DEPTH\_PREDICTION\_FW     | String  | Path to an optionally installed mono depth prediction framework. Please refer to section [Mono depth prediction support](#mono-depth-prediction-support) for details.  Option only available, if support has been installed.     | empty                               |
| DEPTH\_PREDICTION\_MODEL            | String  | Name of the trained mono depth prediction model to be used. If left empty, the `default` model is used internally. Option only available, if support has been installed.                                                         | empty                               |

>**Note:** Camera calibration is a procedure to improve the accuracy of positioning results and mandatory for stereo cameras. For obtaining information about how to calibrate stereo cameras please consult Accuware Support. The calibration of a monocular camera can be done from within the app. It is also possible to calibrate cameras, which are connected via a GStreamer pipe. Even pre-recorded files can be used as input for the calibration. Only WebRTC bound cameras are still forced to use the "old way" (see [Legacy camera calibration of remote cameras](#legacy-camera-calibration-of-remote-cameras)). Whereas in CAM\_MODE `mono` it is allowed to start positioning w/o configured calibration file, a start is rejected in CAM\_MODE `stereo` . The VPS\_SERVER\_URL is a private configuration parameter of the Java property file (`./data/config/dragonfly2.properties`). It is not available for modifications via the JS GUI.

## Startup behavior clarified

If after startup the Java app detects, that there is no Java property file (`./data/config/dragonfly2.properties`), it usually presents an entry form, in which SITE\_ID and SITE\_USERNAME - the only mandatory parameters - are obtained in order to enable the error free start of the app. If a SITE\_PASSWORD is available, it can be entered here too. All credentials are immediately validated against the Accuware Dragonfly Cloud. Validated credentials allow you to use the app for a certain amount of time without the need for an Internet connection. If you don't have credentials, you can directly sign-up from this form for a time limited demo account.

Valid credentials are also required in order to perform the initial download of metadata like floor plans and defined geo fences from the Accuware Dragonfly Cloud.

>**Note:** A validation error of any kind is immediately reported, so that you will not be able to launch the app with invalid credentials. That in turn means, that your initial start of the Java application `unconditionally requires an active Internet connection`.

Once the online validation has been passed, the initial `./data/config/dragonfly2.properties` is written to disk and the Java app launches a GUI automatically into your default browser at `localhost:5000`.

Since this credential input form might not be accessible in headless installations, it is possible to provide a handful of important startup parameters in System.properties via command line parameters.

Those are:

| Parameter     | Meaning                                                                                                          |
|---------------|------------------------------------------------------------------------------------------------------------------|
| siteId        | see SITE\_ID setting                                                                                             |
| login         | see SITE\_USERNAME setting                                                                                       |
| password      | see SITE\_PASSWORD setting, treated as clear text password, will be stored encrypted                              |
| webServerPort | see WEBSERVER\_PORT setting                                                                                      |
| vpsServerUrl  | see VPS\_SERVER\_URL setting                                                                                     |
| logLevel      | see LOG\_LEVEL setting                                                                                           |

If at least `siteId` and `login` are provided at command line level and there is not already a `./data/config/dragonfly2.properties` file available, the values are taken and used to create an initial `./data/config/dragonfly2.properties`, which then can be fine tuned via the configuration GUI in the JS web controlling app or by directly manipulating the file. The credentials are validated against the Accuware Dragonfly Cloud before they are written into the  `./data/config/dragonfly2.properties` file. In case of a failure, the start of the app is abandoned with an error.

>**Note:** The command line parameters are no longer taken into account, if there is a valid `./data/config/dragonfly2.properties`. Please note also, that the GUI is not automatically launched, if the initial configuration of the `./data/config/dragonfly2.properties` has been performed by command line parameters. It is assumed, that intentionally no GUI is necessary, if one launches the app that way.

For example the launch request could look like so now:

```bash
java -jar -DsiteId="1000"                                              \
          -Dlogin="test@test.com"                                      \
          -Dpassword="testpassword"                                    \
          -DwebServerPort="4711"                                       \
          -DvpsServerUrl="https://myVPSServer.myDomain.com/api/v1"     \
          -DlogLevel="1"                                               \
dfja-1.7-dist.jar
```

If the platform validation passes OK, a valid initial `./data/config/dragonfly2.properties` is written and the app launches w/o bringing up the GUI.

## Camera Calibration

The app supports the calibration of **local monocular cameras, cameras bound via a GStreamer pipe or camera recordings available on file**. WebRTC bound cameras are currently not supported. The core functionality is part of the native code and made available via REST API. The JS GUI makes use of it.

The calibration process reads the `CAM_SOURCE` configuration parameter and determines the necessary actions to be taken. As the result a calibration file is created, which needs to be stored in the **./data/config** folder of the app and configured as `CAM_CALIBRATION_FILE` in the settings.

There is a detailed online help available on the `Calibration` page of the Java app, which describes the necessary steps for a successful camera calibration.

## Mono depth prediction support

If optionally selected during installation time the app as of version 1.6 provides a way to connect to a mono depth prediction framework, such as [Monodepth2](https://github.com/nianticlabs/monodepth2). **Monodepth2** is a very nicely working bunch of Python scripts, which use a neural-network to predict the depth information from a **single colored image**, thus simulating a camera equipped with a depth-sensor, like infrared or LIDAR. This allows, at the cost of some computing power, to use a simple monocular camera as if it was a stereo camera in order to overcome monocular SLAM limitations such as forbidden pure-rotations or scale-drift during the mapping process. As to our knowledge it is nowadays the only mono depth prediction framework available (except the abandoned monodepth project, from which it inherits).

To use this new feature, all you need to do is to clone the Monodepth2 package and add the required packages. For license limitations we are not allowed to install Monodepth2 beforehand. Although recommended by the Monodepth2 team, it is not mandatory to use the virtual environment provided by [Anaconda](https://www.anaconda.com/distribution/).

We have found in our tests under **Python 3** (mandatory) that it is sufficient to install the required packages globally using the package managers of the respective operating systems. Please check in the [Backup section](#backup) which steps are necessary for your operating system to install and run Monodepth2.

If the installation of Monodepth2 is done and works, we only need the reference to the installation location of Monodepth2. Therefore we have introduced the new configuration variable `PATH_TO_DEPTH_PREDICTION_FW`. This and two other configuration means (describing the mode and the model to be used) are sufficient to get started with mono depth prediction. Please refer to section [Configuration](#configuration) for details.

Once the app now starts with a configured `PATH_TO_DEPTH_PREDICTION_FW` the required prediction model will be downloaded. This needs to have internet connectivity for at least one time, until the download and configuration is finished.

Please note that due to the high computer load, it makes little sense to use a mono depth prediction framework in systems that neither have a GPU nor where their GPU is not supported by [CUDA](https://developer.nvidia.com/cuda-zone). It will generally work, but a normal CPU will be overwhelmed by the computing capacity required.

To avoid disappointments, please check first if your system is generally suited to benefit from mono depth prediction because [it has the required CUDA support](https://en.wikipedia.org/wiki/CUDA)

## CAM_SOURCE explained

The configuration parameter `CAM_SOURCE` is an important configuration parameter and has a couple of variances. The default value is -1, which advices the app to use the default system camera.

Other numerical values can be used and define the index of the camera as enumerated by the system. A list of available cameras can be obtained by the API function ["Get list of cameras"](#get-list-of-cameras). There can be multiple indexes specified (comma separated, (e.g. left cam index","right cam index), so in case a stereo camera is made of two separate USB cameras.

On the other hand the parameter allows the specification of `GStreamer pipelines` or parameter strings to be used for by `FFMPEG` internally. The prefixes `gstreamer:` and `ffmpeg:` are used to decide, which internal API needs to be fed with the rest of the parameter string. Those pipelines can be used in order to mount external cameras via network or internet.

There is an additionally supported prefix named `webrtc:`, which enables the app to use the video from an external WebRTC source.

It goes without saying, that using GStreamer and/or FFMPEG requires to have the appropriate GStreamer 1.0 and FFMPEG APIs available on your system. This is the case if you have installed the Dragonfly app from the installation procedure [provided here](https://www.accuware.com/support/dragonfly-java-app-software-setup/).

If you intend to use WebRTC via GStreamer, then you need to have at least GStreamer 1.14 installed, better 1.16 since it comes with a lot of improvements for the `libnice` component. There are some restrictions with Ubuntu and higher versions of GStreamer. As a fall-back the Java app supports WebRTC using a Chromium instance, which is configured in the setting `PATH_TO_CHROMIUM`. Please [download the latest Chromium app for your system](https://chromium.woolyss.com/download/en/), install it and configure the path. Example configuration for macOS: `PATH_TO_CHROMIUM=/Applications/Chromium.app/Contents/MacOS/Chromium` or if you are preferring Chrome `PATH_TO_CHROMIUM=/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome`

Some ready to be used examples for pipeline configurations and FFMPEG parameters are given below. You might have to adapt the IP addresses and/or URLs:

### GStreamer configuration to be used in order to consume a Raspberry PI camera, streaming H.264 via TCP (Accuware Dragonfly Car)

The RPI is streaming RTP from 192.168.188.66, port 5000:

```ini
CAM_SOURCE=gstreamer:tcpclientsrc host=192.168.188.66 port=5000 ! gdpdepay !  rtph264depay ! avdec_h264 ! videoconvert ! appsink sync=false
```

> Additional info: [RTP streaming from Raspberry PI](#rtp-streaming-from-raspberry-pi)

GStreamer target sink is always `appsink`.

### GStreamer configuration to be used in order to consume a Raspberry PI camera, streaming H.264 via RTSP

The RPI at 192.168.188.66 is providing an RTSP stream:

```ini
CAM_SOURCE=gstreamer:rtspsrc location=rtsp://192.168.188.66:8554/test latency=0 buffer-mode=auto ! rtph264depay ! avdec_h264 ! videoconvert ! appsink sync=false
```

The PI needs to be available on the same network as the Java app.

> Additional info: [RTSP streaming from Raspberry PI](#rtsp-streaming-from-raspberry-pi)

### GStreamer configuration to be used to consume a web video, provided as RTSP stream

```ini
CAM_SOURCE=gstreamer:rtspsrc location=rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov latency=0 buffer-mode=auto ! rtph264depay ! avdec_h264 ! videoconvert ! appsink sync=false
```

### GStreamer configuration to be used to consume an MJPEG stream

The source, e.g. at 192.168.1.65:5000 should stream the video like so:

```bash
gst-launch-1.0 v4l2src device=/dev/video0 ! 'video/x-raw,width=640,height=480,framerate=30/1' ! decodebin name=dec ! queue ! videoconvert ! jpegenc ! queue ! multipartmux ! tcpserversink host=192.168.1.65 port=5000
```

Then the matching `CAM_SOURCE` would be:

```ini
CAM_SOURCE=gstreamer:tcpclientsrc host=192.168.1.65 port=5000 ! multipartdemux ! image/jpeg, framerate=30/1  ! jpegparse ! jpegdec ! videoconvert  ! appsink sync=false
```

Use this, if the streaming device does not allow live H264 encoding, or if H.264 is not satisfying.

### GStreamer configuration for H.264 streaming from a Linux device

The source, e.g. at 192.168.1.65:5000 should stream the video like so:

```bash
gst-launch-1.0 v4l2src device=/dev/video0 ! 'video/x-raw,width=640,height=480,framerate=30/1' ! videoconvert ! x264enc tune=zerolatency byte-stream=true ! rtph264pay ! gdppay ! tcpserversink host=192.168.1.65 port=5000 sync=false
```

Then the matching `CAM_SOURCE` would be:

```ini
CAM_SOURCE=gstreamer:tcpclientsrc host=127.0.0.1 port=5000  ! gdpdepay !  rtph264depay ! avdec_h264 ! videoconvert ! appsink sync=false
```

Much lower bandwidth requirements than MJPEG, but requires live encoding and some CPUs might not be powerful enough.

### GStreamer configuration for accelerated H.264 streaming from a Linux device

Same as above, but does not require a powerful CPU encoding. However, all platforms are not compatible. Raspberry Pi and Jetson Nano should work fine.

The source, e.g. at 192.168.1.65:5000 should stream the video like so:

```bash
gst-launch-1.0 v4l2src device=/dev/video0 ! 'video/x-raw,width=640,height=480,framerate=30/1' ! videoconvert ! omxh264enc ! rtph264pay ! gdppay ! tcpserversink host=192.168.1.65 port=5000 sync=false
```

Then the matching `CAM_SOURCE` would be:

```ini
CAM_SOURCE=gstreamer:tcpclientsrc host=127.0.0.1 port=5000  ! gdpdepay !  rtph264depay ! avdec_h264 ! videoconvert ! appsink sync=false
```

### FFMPEG configuration to consume a local MP4 video from disk

```ini
CAM_SOURCE=ffmpeg:/home/user/documents/test.mp4
CAM_SOURCE=ffmpeg:/home/user/documents/test.mp4 --startingFrame=500
CAM_SOURCE=ffmpeg:file://home/user/documents/test.mp4
```

### FFMPEG configuration to consume an RTMP video from the web

```ini
CAM_SOURCE=ffmpeg:rtmp://184.72.239.149/vod/mp4:bigbuckbunny_1500.mp4
```

### WebRTC configuration to get the video of a WebRTC enabled Raspberry PI via Accuware Signaling Server

```ini
CAM_SOURCE=webrtc:https://dragonfly-demo.accuware.com:8443/?room=pi&mode=1&use-hw-codec=false
```

This connection string addresses a Raspberry PI (using UV4L WebRTC and Signaling Proxy), who is booked into room `pi`. The connection string advices to immediately connect to the PI (mode=1) and the PI's H.264 hardware codec shall not be used during this session.

> Additional info: [WebRTC streaming from Raspberry PI](#webrtc-streaming-from-raspberry-pi)

### WebRTC configuration to get the video from another PC, booked in into room "test" at the Accuware Signaling Server

```ini
CAM_SOURCE=webrtc:https://dragonfly-demo.accuware.com:8443/?room=test&mode=2
```

The PC is using the [Accuware Dragonfly Demo](https://dragonfly-demo.accuware.com), enters the same room and just waits. Once the camera is opened with the a.m. connection string from the Java app, the browser is contacted (probably you need to allow access to the camera and if auto-answer is not enabled, a bell will ring) and the video is displayed. `mode=2` automatically starts the call to the browser.

### WebRTC configuration to get the video from a DJI drone

```ini
CAM_SOURCE=webrtc:https://dragonfly-demo.accuware.com:8443/licensed/?userid=7773601236&licensekey=D00289AE-DBF2-42C6-ABCD-8CBE81D555D0&mode=2
```

The drone is using an [Android app and a library](https://github.com/accuware/djistreamerlib) in order to enter the licensed section specified by userId and licenseKey and enable the video transmission. It then waits for the call of the Java app in order to deliver the video.

Please also checkout these resources in order to learn about the ways to [connect to DJI drones via WebRTC](https://www.dragonflycv.com/real-time-rtsp-streaming-from-dji-drones/).

### WebRTC configuration to get the video from a Raspberry PI directly from UV4L

```ini
CAM_SOURCE=webrtc:https://192.168.188.36/stream/webrtc/?use-hw-codec=false&server-url=https://dragonfly-demo.accuware.com:8443/uv4l&headless=true
```

If the PI is on the same network as the Java app or publicly reachable, there is no need to incorporate a Signaling Server and Signaling Proxy, since both parties can directly exchange the necessary parameters. The Java app utilizes the [UV4L WebRTC extension protocol](https://www.linux-projects.org/webrtc-signalling/) then.

> Additional info: [WebRTC streaming from Raspberry PI using UV4L directly](#webrtc-streaming-from-raspberry-pi-using-uv4l-directly).

All these example CAM_SOURCE strings need some extra explanations. The first parameter after `webrtc:` is always the URL of the signaling server or the server, which provides WebRTC functionality directly. It can be any URL, as long as the proprietary `Accuware WebRTC Signaling Protocol` (either licensed or community edition) is matched or the described server is providing the UV4L WebRTC protocol via REST API directly.

| Parameter    | Meaning                                                                                | Default                                     |
|--------------|----------------------------------------------------------------------------------------|---------------------------------------------|
| room         | The room in which to meet the other party                                              |                                             |
| mode         | The mode of operation 1: Automatic call to a PI, 2: Automatic call to any other source | 1                                           |
| name         | Optional name, with which you appear at the other party                                |                                             |
| quality      | Quality of JPEG encoding (Chromium only)                                               | 0.75                                        |
| fps          | Target frame rate at JPEG level (Chromium only)                                        | 30                                          |
| headless     | Headless run of the browser (Chromium only), false: non-headless                       | true                                        |
| userId       | UserID for the licensed version of the Accuware Signaling Server                       |                                             |
| licenseKey   | License key for the licensed version of the Accuware Signaling Server                  |                                             |
| use-hw-codec | Enforcement of the Raspberry PI hardware codec (H.264)                                 | Chromium: false, GStreamer: true            |
| server-url   | Required only, if you use the direct UV4L approach                                     | Required only, if `PATH_TO_CHROMIUM` is set |

>**Note:** The `use-hw-codec` parameter addresses a problem with Chromium. We suggest to not enforce PI's H.264 hardware codec with this browser, since H.264 is not supported there. If the remote party is NOT a Raspberry PI the parameter is ignored. For the GStreamer and Chrome approach there is no such a problem with the PI's H.264 hardware encoder, you are free to use the hardware encoder or not.

Unless the direct UV4L approach is used with a Raspberry PI we need a little proxy between our Signaling Server and UV4L, running on the PI. Please contact us for details and support. See [Backup](#backup) section at the end of this document for the various ways to make WebRTC running with a Raspberry PI.

## JConsole monitoring

If enabled, the function of the app can be monitored by using [JConsole](https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html). If the Java app is running, and `JMX_REMOTE_MONITORING_ENABLED` is set to `true` in configuration, run

```bash
jconsole
```

Since an authentication is necessary, the first connection attempt will fail. **Cancel** the error message dialog, that appears.

In the dialog window in foreground now...

- click `Remote Process` and enter `localhost:<port>` in the edit field below, whereas `<port>` is the configured `JMX_RMI_REGISTRY_PORT`, `33985` by default
- enter username and password from `./data/config/jmx_password` and hit **Connect**. Default for username and password is `accuware` and `4711`.
- on the next screen hit **Insecure connection**, because the JMX server doesn't support SSL. You will be able to monitor memory consumption, threads and CPU utilization of the Java app.

## CSV logging

If enabled, the app provides a CSV log of all positions made. This log can be found in the `./data/logs` folder and is named `positionlog.csv`. It has an automatic daily rollover mechanism built in.

The log contains the following info elements, separated by ",". `rxx` parameters are the elements of the rotation matrix, `state` is the current positioning state at time of recording (see [Get status](#get-status))

```html
EPOCH-timestamp-in-milliseconds,siteId,levelId,latitude,longitude,altitude,pitch,yaw,roll,r00,r01,r02,r10,r11,r12,r20,r21,r22,x,y,z,state
```

For example:

```html
549567996549,1005,0,-0.000587,0.014597,-0.022552,-0.032113,-0.184232,0.677457,0.999925,-0.011822,-0.003222,0.011824,0.999930,0.000522,0.003215,-0.000560,0.999995,-8.44,1.21,0.84,NAVIGATION
```

>**Note:** Not all elements must always be available for a given entry. Please also note the coordinate exchange at the metric coordinates x, y and z (explained in the paragraph [Metric coordinates](#metric-coordinates)). Our "Z" is the commonly known "Y" and vice versa.

## Local web server

By default the app exposes a local web server running at port `WEBSERVER_PORT`. You are on it if you can see this GUI.

## REST API

There is no authentication/authorization support on the REST API currently. However, SSL is supported.

All curl based examples are expecting a `WEBSERVER_PORT` that is equal to default 5000. The REST API is fully CORS enabled.

### Summary

Currently 30 API functions plus 1 websocket interface are supported via REST. **None** of these functions have to be called  mandatorily with the exception of [Start positioning](#start-positioning) and [Stop positioning](#stop-positioning), which are the only ways to start/stop mapping/positioning sessions after the Java app has passed the initialization phase. Just a few of these API functions require Internet connectivity. Those are explicitly mentioned.

| API function                                     | Purpose                                                                                                         |
|--------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| **Utilities**                                    |                                                                                                                 |
| GET /api/v1/utils/status                         | [Get status](#get-status)                                                                                       |
| GET /api/v1/utils/errors                         | [Get errors](#get-errors)                                                                                       |
| GET /api/v1/utils/config                         | [Get config](#get-config)                                                                                       |
| PUT /api/v1/utils/config                         | [Save config](#save-config)                                                                                     |
| GET /api/v1/utils/encrypt                        | [Encrypt string](#encrypt-string)                                                                               |
| GET /api/v1/utils/cameras                        | [Get list of cameras](#get-list-of-cameras)                                                                     |
| PUT /api/v1/utils/restart                        | [Restart Java application](#restart-java-application)                                                           |
| PUT /api/v1/utils/terminate                      | [Terminate Java application](#terminate-java-application)                                                       |
| GET /api/v1/utils/updateinfo                     | [Get software update info](#get-software-update-info)                                                           |
| GET /api/v1/utils/calibrationfiles               | [Get calibration files](#get-calibration-files)                                                                 |
| **Positioning**                                  |                                                                                                                 |
| PUT /api/v1/positioning/start                    | [Start positioning](#start-positioning)                                                                         |
| PUT /api/v1/positioning/stop                     | [Stop positioning](#stop-positioning)                                                                           |
| **Mapping (general map information)**            |                                                                                                                 |
| GET /api/v1/manage/maps/list                     | [Get list of maps](#get-list-of-maps)                                                                           |
| GET /api/v1/manage/maps/list/server              | [Get list of maps from server](#get-list-of-maps-from-server)                                                   |
| PUT /api/v1/manage/maps/reset                    | [Reset current map](#reset-current-map)                                                                         |
| **Mapping (virtual markers)**                    |                                                                                                                 |
| GET /api/v1/manage/maps/markers                  | [Get list of virtual markers](#get-list-of-virtual-markers)                                                     |
| GET /api/v1/manage/maps/markers/:id              | [Get virtual marker](#get-virtual-marker)                                                                       |
| POST /api/v1/manage/maps/markers                 | [Create virtual marker](#create-virtual-marker)                                                                 |
| PUT /api/v1/manage/maps/markers/:id              | [Edit virtual marker](#edit-virtual-marker)                                                                     |
| DELETE /api/v1/manage/maps/markers/:id           | [Delete virtual marker](#delete-virtual-marker)                                                                 |
| **Mapping (local operations on a specific map)** |                                                                                                                 |
| POST /api/v1/maps                                | [Save current map](#save-current-map)                                                                           |
| GET /api/v1/maps/:map_name                       | [Load specific map](#load-specific-map)                                                                         |
| DELETE /api/v1/maps/:map_name                    | [Delete specific map](#delete-specific-map)                                                                     |
| PUT /api/v1/maps/:map_name                       | [Update specific map](#update-specific-map)                                                                     |
| **Mapping (map synchronization with server)**    |                                                                                                                 |
| GET /api/v1/manage/maps/list/deltas              | [Get a summary of map deltas between client and server](#get-a-summary-of-map-deltas-between-client-and-server) |
| POST /api/v1/manage/maps/sync                    | [Sync with server](#sync-with-server)                                                                           |
| GET /api/v1/maps/sync                            | [Get general sync status](#get-general-sync-status)                                                             |
| GET /api/v1/maps/sync/:id                        | [Get specific sync status](#get-specific-sync-status)                                                           |
| DELETE /api/v1/maps/sync/:id                     | [Cancel specific sync process](#cancel-specific-sync-process)                                                   |
| **Mapping (visualization)**                      |                                                                                                                 |
| GET /api/v1/manage/maps/mappoints                | [Get list of map points](#get-list-of-mappoints)                                                                |
| **Camera Calibration**                           |                                                                                                                 |
| PUT /api/v1/calibration/start                    | [Start calibration](#start-calibration)                                                                         |
| PUT /api/v1/calibration/stop                     | [Stop calibration](#stop-calibration)                                                                           |
| PUT /api/v1/calibration/flipview                 | [Flip View](#flip-view)                                                                                         |
| PUT /api/v1/calibration/snapshot                 | [Snapshot](#snapshot)                                                                                           |
| PUT /api/v1/calibration/calibrate                | [Calibrate](#calibrate)                                                                                         |

### Utility functions

----

#### Get Status

Obtain status. Result structure is also available as payload of the websocket status response.

##### Request-Resource: GET /api/v1/utils/status

```bash
curl localhost:5000/api/v1/utils/status -X GET
```

##### Response: 200 OK, 'application/json'

```json
{
  "version": "1.7",
  "siteId": "1000",
  "deviceId": "D40K419JW256",
  "state": 1,
  "isStarted": false,
  "mapPointCount": 0,
  "markerCount": 0,
  "loopCount": 0,
  "fps": 0.0
}
```

Complete list of possible object properties. All of these are optional.

| Response-Parameter | Type       | Meaning                                                                                                                                  |
|--------------------|------------|------------------------------------------------------------------------------------------------------------------------------------------|
| version            | String     | The version of the Java app and SDK plus the version of the native code plugin separated by '/'                                          |
| deviceId           | String     | The unique device Id. Can be used in Accuware Dashboard for tracking                                                                     |
| state              | Integer    | The positioning state. 0: Not ready, 1: Idle, 2: Map Initialization, 3: Navigation, 4: Lost, 5: Mapping                                  |
| isStarted          | Boolean    | Whether or not positioning is running                                                                                                    |
| mapPointCount      | Integer    | Mapping: Number of map points detected so far in this session                                                                            |
| markerCount        | Integer    | Mapping: Number of markers detected so far in this session (printed and virtual)                                                         |
| loopCount          | Integer    | Mapping: Number of closed loops detected so far in this session                                                                          |
| fps                | Double     | Camera: Number of frames per second achieved, current value, rounded                                                                     |
| latitude           | Double     | Navigation: Currently known latitude of device (WGS-84)                                                                                  |
| longitude          | Double     | Navigation: Currently known longitude of device (WGS-84)                                                                                 |
| altitude           | Double     | Navigation: Currently known altitude of device (meters)                                                                                  |
| pitch              | Double     | Navigation: Rotation around the X-axis (degrees), see [Pitch, yaw and roll explained](#pitch-yaw-and-roll-explained)                     |
| roll               | Double     | Navigation: Rotation around the Z-axis (degrees)                                                                                         |
| yaw                | Double     | Navigation: Rotation around the Y-axis (degrees)                                                                                         |
| x                  | Double     | Navigation: Metric distance from the origin of the coordinate system along the X-axis in m see [Metric coordinates](#metric-coordinates) |
| y                  | Double     | Navigation: Metric distance from the origin of the coordinate system along the Y-axis in m                                               |
| z                  | Double     | Navigation: Metric distance from the origin of the coordinate system along the Z-axis in m                                               |
| rotationMatrix     | Double[][] | Rotation matrix, see below                                                                                                               |
| levelId            | Integer    | Navigation: Currently known level Id of device                                                                                           |
| currentMapName     | String     | Mapping: Currently used map                                                                                                              |

>**Note:** Depending on the current state, not all response parameter values must be contained in a given response.

### The purpose of the `device_id`

For every site, a license is created. This license allows to use a **limited number of devices** and is valid until an expiration date. Both the maximum number of devices and the expiration date depend on the agreement between Accuware and the owner of the site, and can be consulted on our [online dashboard](https://dashboard.accuware.com/dragonfly/license).

A unique `device_id` is generated for all instances of Dragonfly. This `device_id` remains consistent **as long as the Dragonfly software remains installed at the same location, on the same computer**. Our system automatically checks that the amount of registered device IDs is compliant with the ongoing license to authenticate.

Registering a device is done automatically when running Dragonfly. It requires an Internet connection. At least once within a certain number of days.

If the site owner has to perform modifications on his devices which would imply modification of `device_ids`, it is possible to delete a device from the license once every 24 hours using our [online dashboard](https://dashboard.accuware.com/dragonfly/license). If more changes are required on your license, please contact us at techsupport@accuware.com.

In operation the `device_id` is used for mainly three purposes:

- it is supposed to uniquely identify your running instance of DFJA to the Accuware platform, so that a live-tracking and post-processing of tracking data (e.g. to a spaghetti diagram or heat map) is enabled.
- it is used to control compliance with license conditions.
- it is used to determine the ownership of created and published maps per site (in conjunction with the `SITE_USERNAME`). Please refer to section [Map Synchronization](#map-synchronization) for details.

### Pitch, yaw and roll explained

![Pitch, yaw and roll](https://user-images.githubusercontent.com/30668077/52219245-89000580-289c-11e9-9a71-13e93bad992c.png "Pitch, yaw and roll")

The rotation matrix is provided as an array of arrays of doubles. More precisely

```java
Double[][] rotationMatrix = new Double[3][3]
```

or

```java
[
    [    r00,    r01,    r02 ],
    [    r10,    r11,    r12 ],
    [    r20,    r21,    r22 ]
]
```

### Metric coordinates

In addition to the delivery of WGS84 coordinates (latitude, longitude and altitude) metric coordinates are delivered since v. 1.0. Those coordinates are always relative distances in m to the origin (0,0), if there is no other origin specified. This projection is known as ordinary 2D X/Y diagram. Since the Dragonfly axis names differ from the axis names in such a diagram, the following rules apply:

- take `x` value as **"X"**, positive values right, negative values left of (0,0)
- forget about the `y` value. This value is always equal to the altitude value and cannot be displayed in a 2D diagram
- mentally rename the `z` value to **"Y"**, positive values above, negative values below (0,0)

Since the error of this projection can become relatively high and coordinates may become unusable, if the origin is not close enough to a bounding box of the delivered coordinates, there is a documented way to move the origin to another coordinate at [Start positioning](#start-positioning). The JS GUI makes use of this by providing the `center coordinates and rotation of all available floor plans` to the core. Now x, y and z are relative distances [m] to the center of a given, internally north-aligned floor plan (even if the real geographical rotation of a floor plan is a completely different one).

----

#### Get Errors

Retrieves a list of initialization errors. This API is just providing the errors happened while initializing the Java app part. It does not cover errors happening in the GUI part.
API should not be used out of the context of the WebApp, since it doesn't give a complete picture.

##### Request-Resource: GET /api/v1/utils/status

```bash
curl localhost:5000/api/v1/utils/errors -X GET
```

##### Response: 200 OK, 'application/json'

```json
{
}
```

----

#### Get config

Retrieve config as provided by  `./data/config/dragonfly2.properties` configuration file for the Java app.

##### Request-Resource: GET /api/v1/utils/config

```bash
curl localhost:5000/api/v1/utils/config
```

##### Response: 200 OK, ConfigurationObject, 'application/json'

```json
{
  "site_id": "some_site_id",
  "site_username": "some_user_name",
  "site_password": "some_password_best_encrypted",
  "log_level": 7,
  "webserver_port": 5000,
  "webserver_ssl_supported": false,
  "webserver_keystore_file": "./data/config/keystore.jks",
  "webserver_keystore_password": "some_keystore_password_best_encrypted",
  "webserver_websockets_enabled": true,
  "webserver_authentication_required": false,
  "jmx_remote_monitoring_enabled": true,
  "jmx_rmi_registry_port": 33985,
  "jmx_rmi_server_port": 33986,
  "audible_status_enabled": true,
  "cam_image_width": 640,
  "cam_image_height": 480,
  "cam_preview_option": 1,
  "cam_fov": 0,
  "cam_source": -1,
  "cam_mode": "mono",
  "cam_calibration_file": "mono.json",
  "map_calibration_method": 3,
  "position_upload_interval": 5000,
  "path_to_chromium": "",
  "position_upload_store": false,
  "position_log_enabled": false,
  "map_fusion_enabled": false,
  "video_recording_enabled": true,
  "path_to_depth_prediction_fw": "/path/to/depth_prediction_fw",
  "depth_prediction_model": "default"
}
```

>**Note:** Not all properties must be included in response payload. The content of the response is the same as documented [here](#configuration-of-the-java-app), all lower-case.

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Could not read config"
  ]
}
```

----

#### Save config

##### Request-Resource: PUT /api/v1/utils/config

```bash
curl localhost:5000/api/v1/utils/config -X PUT -d@config_file.json
```

##### Request-Parameter: ConfigurationObject, 'application/json'

>**Note:** The payload of the request is a **ConfigurationObject** as shown in ["Get Config"](#get-config). Except **"site\_id"** and **"site\_username"** none of the parameters is mandatory. Any change of configuration parameters requires a restart of the Java app.

##### Response: 200 OK, ConfigurationObject, 'application/json'

```json
{
  "site_id": "some_site_id",
  "site_username": "some_user_name",
  "site_password": "some_password_best_encrypted",
  "log_level": 7,
  "webserver_port": 5000,
  "webserver_ssl_supported": false,
  "webserver_keystore_file": "./data/config/keystore.jks",
  "webserver_keystore_password": "some_keystore_password_best_encrypted",
  "webserver_websockets_enabled": true,
  "webserver_authentication_required": false,
  "jmx_remote_monitoring_enabled": true,
  "jmx_rmi_registry_port": 33985,
  "jmx_rmi_server_port": 33986,
  "audible_status_enabled": true,
  "cam_image_width": 640,
  "cam_image_height": 480,
  "cam_preview_option": 1,
  "cam_fov": 0,
  "cam_source": -1,
  "cam_mode": "mono",
  "cam_calibration_file": "mono.json",
  "map_calibration_method": 3,
  "position_upload_interval": 5000,
  "path_to_chromium": "",
  "position_upload_store": false,
  "position_log_enabled": false,
  "map_fusion_enabled": false,
  "video_recording_enabled": true,
  "path_to_depth_prediction_fw": "/path/to/depth_prediction_fw",
  "depth_prediction_model": "default"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Configuration could not be saved"
  ]
}
```

----

#### Encrypt string

Function provides encrypted version of a string provided as `value` in query string. Result can be used everywhere in config, where encrypted values are accepted.

##### Request-Resource: GET /api/v1/utils/encrypt

```bash
curl localhost:5000/api/v1/utils/encrypt -G --data-urlencode "value=password"
```

##### Request-Parameter: Query-String

| Parameter | Meaning                  | Default |
|-----------|--------------------------|---------|
| value     | The string to be encoded | n.a.    |

##### Response: 200 OK, 'text/html'

```html
5D6D8A590CB4BA7E07F8FF84E7EF45E0
```

----

#### Get list of cameras

Function delivers a list of available cameras. The position of each listed camera can be used as parameter "CAM_SOURCE" in configuration, if a specific camera should be selected.

>**Note:** The list of available cameras is not a static list. It changes with the addition/removal of cameras.

##### Request-Resource: GET /api/v1/utils/cameras

```bash
curl localhost:5000/api/v1/utils/cameras -X GET
```

##### Response: 200 OK, 'application/json'

```json
[
  "HD Pro Webcam C920 0x14100000046d082d",
  "FaceTime HD Camera DJH4273WRR4F6VTDQ"
]
```

----

#### Restart Java application

Restart Java application. Does work only, if the Java app is launched as JAR (not from IDE)

##### Request-Resource: PUT /api/v1/utils/restart

```bash
curl localhost:5000/api/v1/utils/restart -X PUT
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Could not restart Java application"
  ]
}
```

----

#### Terminate Java application

Terminate Java application. Does work only, if the Java app is launched as JAR (not from IDE)

##### Request-Resource: PUT /api/v1/utils/terminate

```bash
curl localhost:5000/api/v1/utils/terminate -X PUT
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Could not terminate Java application"
  ]
}
```

----

#### Get software update info

Clients may obtain information about the availability of a new software bundle version. Internally the Java app checks at startup and once per five minutes if there is a new software bundle version available server side. In case it is, the info is traced using the INFO tag to the console:

```html
2019-02-18 13:35:03,841 SoftwareUpdateChecker$1$1 71 INFO  [I/O dispatcher 2]: Dragonfly2SDK:  New application version available: Version: x.xx, Linux download: https://s3.amazonaws.com/navizon.indoors.camera/dragonfly/dragonfly_update_linux_yyyymmdd.zip, macOS download: https://s3.amazonaws.com/navizon.indoors.camera/dragonfly_update_mac_yyyymmdd.zip
```

In parallel the GUI checks once per minute, if an update is available. In case it is, an alert is popping up once per session, which recommends you to update.

##### Request-Resource: GET /api/v1/utils/updateinfo

```bash
curl localhost:5000/api/v1/utils/updateinfo -X GET
```

##### Response: 200 OK, 'application/json'

```json
{
  "version": "x.xx",
  "patch_url_linux": "https://s3.amazonaws.com/navizon.indoors.camera/dragonfly/dragonfly_update_linux_yyyymmdd.zip",
  "patch_url_macos": "https://s3.amazonaws.com/navizon.indoors.camera/dragonfly/dragonfly_update_mac_yyyymmdd.zip"
}
```

##### Response: 304 Not Modified

There is no new version available

#### Get calibration files

This API function has been added for convenience only. It has just a practical meaning for the JS GUI by enabling it to retrieve the names of currently available camera calibration files, which are by default all files ending with `.json` in the `./data/config` directory. This helps preventing unnecessary typo errors while configuring and/or switching between calibration files.

##### Request-Resource: GET /api/v1/utils/calibrationfiles

```bash
curl localhost:5000/api/v1/utils/calibrationfiles -X GET
```

##### Response: 200 OK, 'application/json'

```json
[
  "dji-mavic-pro-calibration-v3.json",
  "iradar_stereo.json",
  "logitech.json",
  "right.json",
  "rpi-fisheye-v2.json",
  "rpi-fisheye-v3.json",
  "rpi-fisheye.json",
  "rpi-non-fisheye.json",
  "rpi1_classic.json"
]
```

### Positioning

Function provides means to start and stop a positioning session. By using `dontmap` you advise the software to no longer accumulate mapping data, but navigate on the existing map.

----

#### Start positioning

##### Request-Resource: PUT /api/v1/positioning/start

```bash
curl localhost:5000/api/v1/positioning/start -X PUT
```

```bash
curl localhost:5000/api/v1/positioning/start&dontmap=true -X PUT
```

##### Request-Parameter: Query-String

| Parameter | Meaning                                                                     | Default |
|-----------|-----------------------------------------------------------------------------|---------|
| dontmap   | **true**: Disables simultaneous mapping, only navigating on an existing map | false   |

##### Request-Parameter: Payload, 'application/json'

| Parameter | Meaning                                                                                                                                                                                                                         | Default |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| array     | Optional array of objects, containing the Integer `levelId` and Double WGS84 coordinates of the origin for the metric projection (`origin_lat, origin_lng, origin_alt` plus the Double `rotation` of the floor plan in degrees) | n.a.    |

```json
[
  {
    "levelId": 1,
    "origin_lat": 25.xxxxxx,
    "origin_lng": -80.xxxxxx,
    "origin_alt": 0.0,                              // always 0, ignored
    "rotation": 170.0
  },
  {
    ...
  }
]
```

The JS GUI is providing this array by specifying the center coordinates of all available floor plans of all possible levels. This generally allows proper metric coordinates, it is not required for ordinary WGS84 projection.

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Could not open camera"
  ]
}
```

----

#### Stop positioning

##### Request-Resource: PUT /api/v1/positioning/stop

```bash
curl localhost:5000/api/v1/positioning/stop -X PUT
```

##### Result: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

### Mapping

Functions to list, load, save, edit, reset and delete maps locally plus functions to sync client and server maps.  

----

#### Get list of maps

##### Request-Resource: GET /api/v1/manage/maps/list

```bash
curl localhost:5000/api/v1/manage/maps/list -X GET
```

##### Result: 200 OK, 'application/json'

```json
[
  {
    "map_name": "map1",
    "description": "Room1",
    "checksum": "30b0f2e646e4373c2042de216d9a46be9778b775",
    "status": 1,
    "version": 1,
    "timestamp": 1515495422,
    "owner": "me@accuware.com",
    "device_id": "D9P0JOWQ9B9J",
    "access": 0
  },
  {
    "map_name": "map2",
    "description": "Room2",
    "checksum": "f7b6bd202b20d6822d4e36d27a9853ef381bdadf",
    "status": 1,
    "version": 5,
    "timestamp": 1515495422,
    "owner": "me@accuware.com",
    "device_id": "D9P0JOWQ9B9J",
    "access": 0
  }
]
```

| Response-Parameter | Type    | Meaning                                        |
|--------------------|---------|------------------------------------------------|
| map_name           | String  | The name of the map                            |
| description        | String  | The description of the map                     |
| checksum           | String  | 40 byte SHA-1 checksum over the map content    |
| status             | Integer | Status of the map, see below                   |
| version            | Integer | Version number, up-counted on every change     |
| timestamp          | Integer | EPOCH timestamp in seconds of last change      |
| owner              | String  | Accuware Username of the owner of the map      |
| device_id          | String  | ID of the device, on which the map was created |
| access             | Integer | Access control flag (not used yet)             |

| status           | Value | Meaning                                  |
|------------------|-------|------------------------------------------|
| NOT_SYNCHRONIZED | 0     | Client map, not synchronized with server |
| SYNCHRONIZED     | 1     | Client and server map identical          |

----

#### Get list of maps from server

An optional function to obtain a list of maps held by the server for informational purposes. This API function requires an Internet connection.

With the exception of the missing **status** property the payload returned is identical to the [Get list of maps](#get-list-of-maps) API.

##### Request-Resource: GET /api/v1/manage/maps/list/server

```bash
curl localhost:5000/api/v1/manage/maps/list/server -X GET
```

##### Result: 200 OK, 'application/json'

```json
[
  {
    "map_name": "map3",
    "description": "Room3",
    "checksum": "30b0f2e646e4373c2042de216d9a46be9778b775",
    "version": 10,
    "timestamp": 1515495422,
    "owner": "you@accuware.com",
    "device_id": "D9P0JOWQ9B8X",
    "access": 0
  }
]
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No Internet access."
  ]
```

----

#### Reset current map

Resets the currently used map. A running positioning session will be terminated.

##### Request-Resource: PUT /api/v1/manage/maps/reset

```bash
curl localhost:5000/api/v1/manage/maps/reset -X PUT
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

----

#### Save current map

Put the current map results into a map named by `map_name` and give it a `description`. Both request payload parameters are mandatory and pre-processed. Only alphanumeric characters and some special selected characters are allowed (see regular expression below). The length of both parameters is limited. Both parameters are lower-cased, spaces are transformed to `_`.

##### Request-Resource: POST /api/v1/maps

```bash
curl localhost:5000/api/v1/maps -X POST -H "Content-Type: application/json" -d '{"map_name": "map1", "description":"Map 1"}'
```

##### Request-Parameter: Payload, 'application/json'

| Parameter | Meaning                                                                                                                                              | Default |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| object    | Object containing mandatory `map_name` and `description` as strings. Allowed characters must match this regular expression: `^[0-9a-zA-Z\ _\-\.@]+$` | n.a.    |

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "The system has not computed a map yet."
  ]
}
```

----

#### Load specific map

##### Request-Resource: GET /api/v1/maps/:map_name

```bash
curl "localhost:5000/api/v1/maps/map1" -X GET
```

##### Request-Parameter: URL-Parameter, urlencoded

| Parameter | Meaning                                            | Default |
|-----------|----------------------------------------------------|---------|
| :map_name | Mandatory name of the map to be loaded, urlencoded | n.a.    |

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

Map has been loaded and will be used for positioning.

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "The system could not load the map. Does it exist?"
  ]
}
```

----

#### Delete specific map

Deletes the map named by parameter `:map_name`.

##### Request-Resource: DELETE /api/v1/maps/:map_name

```bash
curl localhost:5000/api/v1/maps/map1 -X DELETE
```

##### Request-Parameter: URL-String

| Parameter | Meaning                                 | Default |
|-----------|-----------------------------------------|---------|
| :map_name | Mandatory name of the map to be deleted | n.a.    |

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Map \"map1\" doesn\u0027t exist."
  ]
}
```

----

#### Update specific map

##### Request-Resource: PUT /api/v1/maps/:map_name

Allows to change `map_name` and/or `description` of the map named by parameter `:map_name`.

```bash
curl localhost:5000/api/v1/maps/map1 -X PUT -H "Content-Type: application/json" -d '{"map_name": "map1", "description": "New Description"}'
```

##### Request-Parameter: URL-Parameter, urlencoded

| Parameter | Meaning                                 | Default |
|-----------|-----------------------------------------|---------|
| map_name  | Mandatory name of the map to be deleted | n.a.    |

##### Request-Parameter: Payload, 'application/json'

| Parameter | Meaning                                                                                                                                              | Default |
|-----------|------------------------------------------------------------------------------------------------------------------------------------------------------|---------|
| object    | Object containing mandatory `map_name` and `description` as strings. Allowed characters must match this regular expression: `^[0-9a-zA-Z\ _\-\.@]+$` | n.a.    |

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Map \"map1\" doesn\u0027t exist."
  ]
}
```

----

### Map Synchronization

The purpose of the **Map Synchronization** process is to achieve equality of the maps held locally (on client) and remotely (on server). On client machines maps are located in the sub directory `./data/maps`, grouped by `Accuware Site Id`. The folders contain zero or more map files (extension `.osm`) and a `maplist.json` file, which is used to hold meta data. It is not recommended to directly manipulate either of these files.

The structure below already shows, that the map name must be unique throughout an Accuware Site. The Java app is maintaining a File System Watcher, being triggered by either changes of map meta data (e.g. map_name or description). If Internet access is available at this moment, the Java app internally utilizes the [Get a summary of map deltas between client and server](#get-a-summary-of-map-deltas-between-client-and-server) API in order to determine the synchronization state between client and server. It reports the results immediately via the websocket status API to the dashboard, which visualizes the results. Version 1.7 just indicates, if a [Sync with server](#sync-with-server) request should be issued or not.

In addition the dashboard itself tries once per minute to determine the state to bring it to the user's attention.

```html
.
├── 1000
│   ├── maplist.json
│   └── test.osm
└── 1005
    ├── maplist.json
    └── test.osm

2 directories, 4 files
```

For the function of the Dragonfly app it is completely sufficient to work with local maps only. Just in case you want to share maps with third parties or do a collaborative creation, utilization or maintenance of maps a synchronization process is necessary, which synchronizes the maps available locally with the maps available server side for a particular Accuware Site.

The REST API [Sync with server](#sync-with-server) is intended to provide means to start such a synchronization process.  

----

#### Basic rules

There are some rules, enforced by the REST server while synchronizing against the Accuware Servers.

- There is a `maplist.json` file for each site, maintained by the app, which has a central role for the synchronization. If the file gets corrupted or lost, all knowledge of metadata for the available local maps (owner, device_id, version, access, description etc.) is lost. This file will be recreated then. Found local maps will be renamed using arbitrary random names and the description `Lost_and_Found`, with version 1, the current user as owner and the current device as device_id in order to prevent future synchronization problems. This mechanism btw also applies to maps, which you just simply have copied into the `maps` directory of the site.
- Maps belong to the `owner`, which is the currently used `Accuware Site Username` on creation of a map for a given site. The `device_id` is an additional discrimination element, allowing one and the same user to manage maps for a site from different machines without synchronization problems.
- Anybody is allowed to update metadata of a map and - more importantly - to contribute to the map content by mapping.
- Renaming a map locally creates a new map with the old content. This in turn leads to the removal of the map from the server, if you are the owner of the device and the map was created on this device. If you are not the owner or are working from a different device, a new map with the old content will be created on the server, with you as `owner` and your device as `device_id` on next synchronization. At the same time the old map will be restored on your machine, since it is still existent on the server.
- Only `owners` working from the device identified by `device_id` are allowed delete a map from the server and by that subsequently from all clients on synchronization.

More generally these **Synchronization rules** apply, which in fact simply and completely describe, what happens on synchronization per site:

| Map exists           | Synchronization rule                                                                                                             |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------|
| On client only       | Maps owned by the current user and created on current device will be **created on server**, otherwise **deleted on client**.     |
| On server only       | Maps owned by the current user and created on the current device will be **deleted on server**, otherwise **created on client**. |
| On server and client | Maps with a higher version on client will be **updated on server**, otherwise **updated on client**.                             |

#### Get a summary of map deltas between client and server

This API can be used in order to get a detailed overview about what is available

- on client only
- on server only
- on client and server

The function internally determines the required actions and displays it. This API is just informational and used internally by the Java app. It is not necessary at all for the synchronization request.

In order to proceed successfully an Internet connection is required.

##### Request-Resource: GET /api/v1/manage/maps/list/deltas

```bash
curl "localhost:5000/api/v1/maps/list/deltas" -X GET
```

##### Result: 200 OK, 'application/json'

```json
{
  "status": "success",
  "clientOnly": {
    "createOnServer": [],
    "deleteOnClient": []
  },
  "serverOnly": {
    "createOnClient": [],
    "deleteOnServer": []
  },
  "clientAndServer": {
    "updateOnClient": [],
    "updateOnServer": []
  }
}
```

| Response-Parameter | Type            | Meaning                                                         |
|--------------------|-----------------|-----------------------------------------------------------------|
| status             | String          | Either "error" or "success"                                     |
| clientOnly         | String          | Container object for maps existing on client only               |
| serverOnly         | String          | Container object for maps existing on server only               |
| clientAndServer    | String          | Container object for maps existing on client and server         |
| createOnServer     | Array of String | List of map names, which will be created on server on next sync |
| deleteOnClient     | Array of String | List of map names, which will be deleted on client on next sync |
| createOnClient     | Array of String | List of map names, which will be created on client on next sync |
| deleteOnServer     | Array of String | List of map names, which will be deleted on server on next sync |
| updateOnClient     | Array of String | List of map names, which will be updated on client on next sync |
| updateOnServer     | Array of String | List of map names, which will be updated on server on next sync |

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No Internet access."
  ]
```

#### Sync with server

The synchronization is always a client-initiated process using this API. Because it might need a lot of time (please note: big map files probably have to be up- and downloaded), this function just initiates a synchronization process. In case, no other sync is currently running, it returns immediately, providing an ID, which furthermore can be used to obtain status and result or to cancel a sync process.

##### Request-Resource: POST /api/v1/manage/maps/sync

```bash
curl localhost:5000/api/v1/manage/maps/sync -X POST
```

##### Response: 200 OK, 'application/json'

```json
{
  "id": 1
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Sync process with id 1 already running."
  ]
}
```

#### Get general sync status

This API call can be used in order to obtain the status of synchronization process.  

##### Request-Resource: GET /api/v1/maps/sync

```bash
curl "localhost:5000/api/v1/maps/sync" -X GET
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "not running"
}
```

#### Get specific sync status

Obtains the status of a specific synchronization process, identified by parameter `:id`, previously returned from a  [Sync with server](#sync-with-server) API call.

##### Request-Resource: GET /api/v1/maps/sync/:id

```bash
curl "localhost:5000/api/v1/maps/sync/1" -X GET
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "finished successfully"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No sync process with id 1 running."
  ]
}
```

#### Cancel specific sync process

Can be used in order to cancel a specific synchronization process, identified by parameter `:id`, previously returned from a  [Sync with server](#sync-with-server) API call. This call just abandons a synchronization, it doesn't roll back the results, so the final state might be something "in between" and would probably need another sync request to return to a stable situation.

##### Request-Resource: DELETE /api/v1/maps/sync/:id

```bash
curl "localhost:5000/api/v1/maps/sync/1" -X DELETE
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "cancelled"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No sync process with id 1 running."
  ]
}
```

### Virtual marker management

Virtual marker operations do always work on the currently loaded and/or used map. A positioning session must be running and in order to make the most functions proceed, the positioning status must have been reached and kept.

----

#### Get list of virtual markers

Gets a list of IDs of virtual markers. The IDs can be used in subsequent API calls.

##### Request-Resource: GET /api/v1/manage/maps/markers

```bash
curl localhost:5000/api/v1/manage/maps/markers
```

##### Response: 200 OK, 'application/json'

```json
[
  504,
  529
]
```

----

#### Get virtual marker

Get information for the virtual marker for ID `:id`

##### Request-Resource: GET /api/v1/manage/maps/markers/:id

```bash
curl localhost:5000/api/v1/manage/maps/markers/504
```

##### Response: 200 OK, 'application/json'

```json
{
  "latitude": 2.5958528417757048E-5,
  "longitude": -3.495204476621969E-5,
  "altitude": 0.0,
  "levelId": -11
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No reference point data for id 504"
  ]
}
```

----

#### Create virtual marker

Create virtual marker at position.

##### Request-Resource: POST /api/v1/manage/maps/markers

```bash
curl localhost:5000/api/v1/manage/maps/markers -X POST -H "Content-Type: application/json" -d '{"latitude": 0.0, "longitude": 0.0, "altitude": 0.0, "levelId": 0}'
```

##### Response: 200 OK, 'application/json'

Returns the ID of a newly created marker or -1

```json
1492
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Could not add reference point. Positioning must be running and your device must not be in \u0027Lost\u0027 state."
  ]
}
```

----

#### Edit virtual marker

Edit virtual marker with ID `:id`

##### Request-Resource: PUT /api/v1/manage/maps/markers/:id

```bash
curl localhost:5000/api/v1/manage/maps/markers/1492 -X PUT -H "Content-Type: application/json" -d '{"latitude": 0.0, "longitude": 0.0, "altitude": 1.0, "levelId": 0}'
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "Could not edit reference point. Positioning must be running and your device must not be in \u0027Lost\u0027 state."
  ]
}
```

----

#### Delete virtual marker

Deletes virtual marker with ID `:id`

##### Request-Resource: DELETE /api/v1/manage/maps/markers/:id

```bash
curl localhost:5000/api/v1/manage/maps/markers/1492 -X DELETE
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

### Visualization of mappoints

This API is providing useful results only under special conditions:

- The configuration parameter MAP\_CALIBRATION\_METHOD must be set to 0
- A mapping and positioning session must be running

The results are used internally to visualize the quality of mapping.

----

#### Get list of map points

Gets a list of map points.

##### Request-Resource: GET /api/v1/manage/maps/mappoints

```bash
curl localhost:5000/api/v1/manage/maps/mappoints
```

##### Response: 200 OK, 'application/json'

```json
[
  0.4075704,
  0.14189786,
  0.52664083,
  -0.3725887,
  -0.18158877,
  0.26187122,
  -0.27079827,
  -0.24362804,
  0.6445207,
  0.1305532,
  0.39491752,
  ....
]
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "This API is available only if MAP_CALIBRATION_METHOD is configured to be 0"
  ]
}
```

----

### Calibration

Function provides means to start and stop a calibration session. There are some pre-requirements to be met and calibration instructions necessary to be known. Please find all that on the `Calibration` tab of the app.

At the moment the calibration supports only monocular cameras. Stereo cameras are not supported yet as well as cameras, available via WebRTC.

**Video and status** returned from the calibration engine are made available via websockets (see [Websocket usage](#websocket-usage)).

----

#### Start calibration

##### Request-Resource: PUT /api/v1/calibration/start

```bash
curl localhost:5000/api/v1/calibration/start -X PUT -H "Content-Type: application/json" -d '{"hdist": 178, "useAssistant": true, "useFisheyeModel": false, "useResolution": [640,480]}'
```

##### Request-Parameter: Payload, 'application/json'

| Parameter       | Meaning                                                                                                     | Default |
|-----------------|-------------------------------------------------------------------------------------------------------------|---------|
| hdist           | MANDATORY: HDIST value as measured from the calibration pattern (see online help) in millimeter as Integer. | 178     |
| useAssistant    | OPTIONAL: Boolean, advice the core to use assisted calibration mode (see online help).                      | true    |
| useFisheyeModel | OPTIONAL: Boolean, advice the core to apply fisheye model. Relevant for assisted mode only.                 | false   |
| useResolution   | OPTIONAL: Array of two integers, describing width and height of the video to be obtained from the camera    | n.a.    |

The display of the video within the app is always 640 x 480. For any other input resolution different from 4:3 there might be display issues with respect to aspect ratio.

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "A calibration session is already running"
  ]
}
```

----

#### Stop calibration

##### Request-Resource: PUT /api/v1/calibration/stop

```bash
curl localhost:5000/api/v1/calibration/stop -X PUT
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No calibration process running"
  ]
}
```

----

#### Flip View

This API allows you to mirror the returned video. This might the correct positioning of the calibration pattern easier.

##### Request-Resource: PUT /api/v1/calibration/flipview

```bash
curl localhost:5000/api/v1/calibration/flipview -X PUT
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No calibration process running"
  ]
}
```

----

#### Snapshot

This API is only required in "non-assisted" mode. You need to provide snapshots to the system manually. In assisted mode snapshots are taken automatically, if a good match is detected.

##### Request-Resource: PUT /api/v1/calibration/snapshot

```bash
curl localhost:5000/api/v1/calibration/snapshot -X PUT
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

This status message only provides an information about the successful delivery of the `snapshot` command to the calibration engine. The result of the operation itself is communicated via websockets (see [Websocket usage](#websocket-usage)).
If the image wasn't good enough, there will not even be a reaction of the core.

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No calibration process running"
  ]
}
```

----

#### Calibrate

This API is only required in "non-assisted" mode. You need to start the calibration process manually.

##### Request-Resource: PUT /api/v1/calibration/calibrate

```bash
curl localhost:5000/api/v1/calibration/calibrate -X PUT
```

##### Response: 200 OK, 'application/json'

```json
{
  "status": "success"
}
```

This status only provides the information of the successful delivery of the `calibrate` command to the core. The result of the operation itself is communicated via websockets (see [Websocket usage](#websocket-usage)).

##### Response: 400 Bad Request, 'application/json'

```json
{
  "status": "error",
  "errors": [
    "No calibration process running"
  ]
}
```

----

### Websocket usage

As convenience method for obtaining general and map synchronization status in real time the REST web server provides a websocket based status push service. The URL is `ws://localhost:5000/websockets/status`. The provided information matches generally the response payload of the [Get Status](#get-status) and the [Get a summary of map deltas between client and server](#get-a-summary-of-map-deltas-between-client-and-server) REST API functions. However, due to the necessary discrimination between different payload types, an additional context identifier has been added (see samples below).

The websocket is also used to communicate video and calibration status from the calibration engine to the user.

In case SSL is enabled on the web server, use `wss://localhost:5000/websockets/status`

>**Note:** `WEBSERVER_WEBSOCKETS_ENABLED` must be true in `./data/config/dragonfly2.properties` and the Java app must have started a positioning session. There is currently no websocket status update outside a positioning session (see  [Positioning](#positioning)), use ordinary [Get Status](#get-status) in this case.

There are several ways to contact the web server for a websocket connection:

----

#### 1. The CURL approach

```bash
curl --include \
     --no-buffer \
     --header "Connection: Upgrade" \
     --header "Upgrade: websocket" \
     --header "Host: localhost:5000" \
     --header "Origin: http://localhost:5000" \
     --header "Sec-WebSocket-Key: SGVsbG8sIHdvcmxkIQ==" \
     --header "Sec-WebSocket-Version: 13" \
http://localhost:5000/websockets/status
```

>**Note:** The console will show the raw websocket output

There are currently four different payloads carried via the websocket connection:

- General status information
- Map delta information
- Image information (in case `CAM_PREVIEW_OPTION` is `Preview enabled`)
- Position correction information (in case of loop closing). Internal, not specially documented.

An example payload for a **General status** report looks like so:

>**Note:** Depending on the current state, `status` may contain not all shown properties or even more.

```json

}?~?{
  "status": {
      "version": "1.7/2.1-b19",
      "siteId": "1000",
      "deviceId": "D40K419JW256",
      "state": 2,
      "isStarted": true,
      "mapPointCount": 0,
      "markerCount": 0,
      "loopCount": 0,
      "fps": 20.2429141998291
    }
}?~?{
  "status": {
      "version": "1.7/2.1-b19",
      "siteId": "1000",
      "deviceId": "D40K419JW256",
      "state": 2,
      "isStarted": true,
      "mapPointCount": 0,
      "markerCount": 0,
      "loopCount": 0,
      "fps": 20.2429141998291
    }
}?~?{
  "status": {
      "version": "1.7/2.1-b19",
      "siteId": "1000",
      "deviceId": "D40K419JW256",
      "state": 2,
      "isStarted": true,
      "mapPointCount": 0,
      "markerCount": 0,
      "loopCount": 0,
      "fps": 20.2429141998291
    }
    ...
```

An example **Map Delta** report (see [Map Synchronization](#map-synchronization)) is indicated like so:

```json
}?~({
  "deltas": {
    "status": "success",
    "clientOnly": {
      "createOnServer": ["map1"],
      "deleteOnClient": []
    },
    "serverOnly": {
      "createOnClient": [],
      "deleteOnServer": []
    },
    "clientAndServer": {
      "updateOnClient": [],
      "updateOnServer": []
    }
  }
}
```

For a description of the content see [Get a summary of map deltas between client and server](#get-a-summary-of-map-deltas-between-client-and-server) REST API.

An example payload for an **Image** report looks like so:

```json
}?~({
  "image": {
    ...Base64 encoded JPG image payload...
  }
}
```

Since the payload is BASE64 JPG it can directly been put into an HTML image tag (e.g. with ReactJS):

```json
<img id="videoPreview" alt="Video Preview scaled down to 320x240" width="320" height="240" src={"data:image/jpg;base64," + this.state.image} />
```

Calibration image is provided as MotionJPEG in the element `cb-image`:

```json
}?~({
  "cb-image": {
    ...Base64 encoded JPG image payload...
  }
}
```

The real-time calibration status is transported via the `cb-status` element, which carries a JSON string, which needs to be parsed into an object:

```json
}?~({
    "cb-status": "{\n    \"c\": \"success\",\n    \"ii\": 100,\n    \"it\": \"Calibration not ready to be performed yet. Not enough snapshots have been collected.\",\n    \"ni\": 0,\n    \"op\": \"cs\"\n}",
}
```

| Parameter | Meaning                                                                                                                          |
|-----------|----------------------------------------------------------------------------------------------------------------------------------|
| c         | String, result code, "success" or "error"                                                                                        |
| ii        | Integer, info integer status: 100: Not ready to calibrate,  200: Ready, 300: calibration completed, 400: calibration in progress |
| it        | String, info textual status                                                                                                      |
| ni        | Integer, number of images taken so far                                                                                           |
| op        | String, operation, "calibration status"                                                                                          |
| cj        | String, JSON string, "calibration json", the result of the calibration (only with ii 300)                                        |

----

#### 2. The Chrome approach

There is a variety of Chrome plugins supporting websockets. One of them is [this](https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo). Install and enter the above mentioned status websocket URL into the address field.

## More info

Find usage information and more check [here](https://www.accuware.com)

This documentation [is also available online as secret gist](https://gist.github.com/accuware/dee72abcd94bda40ee14cc3d81a2d9cb).

## Backup

### WebRTC details

With WebRTC

- we can literally "mount" the camera of a Raspberry PI by either directly interacting with the UV4L server on the PI (possible, if the PI is on the same network or publicly available) or by utilizing the signaling server functions of the [**Accuware WebRTC Demo**](https://dragonfly-demo.accuware.com:8443) server on the web, which would allow us to reach any PI in the world, as long as there is a possible path through the Internet,
- we can literally mount any WebRTC enabled device or browser, as long as it is enable to talk to our signaling server. The protocol is simple, we provide a description on demand. This includes DJI drones. By use of the controlling Android app (iOS on demand) we can "WebRTC" the real-time video of a DJI drone to any consumer at any place in the world with latencies below 1 second.

Please have a look at the online help of the [**Accuware WebRTC Demo**](https://dragonfly-demo.accuware.com:8443) server and learn about the principles of WebRTC operation. Try it out. There is also a licensed version, which allows you to test the DJI drone connection, if you are an Android developer ([**Accuware WebRTC Demo, Licensed Edition**](https://dragonfly-demo.accuware.com:8443/licensed))

Details on how to use a WebRTC bound camera within the Dragonfly Java app can be found in section ["CAM_SOURCE explained"](#cam_source-explained).

### RTP streaming from Raspberry PI

[Accuware Gist](https://gist.github.com/accuware/313111f403e041c962c2343dc70ef308)

### RTSP streaming from Raspberry PI

[Accuware Gist](https://gist.github.com/accuware/269a162badfc8e75f444a142b5e0a36a)

### WebRTC streaming from Raspberry PI

[Accuware Gist](https://gist.github.com/accuware/c67ea205c713fb465adaeb0e506d8f7f)

### WebRTC streaming from Raspberry PI using UV4L directly

[Accuware Gist](https://gist.github.com/accuware/370c3fdd758b5cb4b41c6aa2acfe9ce6)

### Create and deploy a self-signed certificate for the Raspberry PI

[Accuware Gist](https://gist.github.com/accuware/b9248e1d2ee8e6e4022557234b7b42a9)

### Legacy camera calibration of remote cameras

For a certain time calibration support was only available on the [Internet, (section Calibration Mode)](https://dragonfly-demo.accuware.com). In the meantime the Java app contains the calibration support, so that for the most use cases the internet solution is no longer required. With one exception: Since the local calibration does not yet support the calibration of **cameras connected via WebRTC**, you have to use the internet solution, if you intend to calibrate one of those cameras. Just copy the entire CAM_SOURCE configuration string after and including `webrtc:` and provide this als `video-url` parameter to the web based calibration app. The internet solution does still support all the other camera connection options, now mostly covered by the app.

Examples:

WebRTC via Signaling Server:

```http
https://dragonfly-demo.accuware.com/?video-url=webrtc:https://dragonfly-demo.accuware.com:8443/?room=pi&mode=1&use-hw-codec=false
```

WebRTC via UV4L Server directly:

```http
https://dragonfly-demo.accuware.com/?video-url=webrtc:https://192.168.188.36/stream/webrtc&use-hw-codec=false
```

Please mind, that you would need to provide SSL support for the UV4L server for being able to calibrate the camera and/or use the browser based approach (`PATH_TO_CHROMIUM` is set) for positioning operations (`mixed content` policy of the browsers).

An RTSP source needs to be **available on the public Internet** and is configured using their public URL:

```http
https://dragonfly-demo.accuware.com/?video-url=rtsp://my-public-rtsp-server:8554/test
```

If it is not possible to publish your RTSP source on the Internet, not even for the short time of calibration, then there is a little [Node helper project](https://github.com/accuware/gstreamer), which can act as an `RTSP-to-MJPEG converter`. Your RTSP source would then be made available to the calibration server using this URL:

```http
https://dragonfly-demo.accuware.com/?video-url=https://localhost:9000
```

The **localhost:9000** link has to be secure because of the `mixed content` policy of the browsers.

This proxy, BTW, would also help to calibrate a Raspberry PI camera from a PI, which streams just RTP via TCP (see [RTP streaming from Raspberry PI](#rtp-streaming-from-raspberry-pi)).

### Monodepth2 installation on Linux

- Clone Monodepth2 repository

```bash
git clone https://github.com/nianticlabs/monodepth2.git
```

- Install Python3 and some common Python3 tools

```bash
sudo apt install python3 python3-dev python3-numpy python3-pip
```

- Install Python dependencies for Monodepth2

```bash
sudo pip3 install numpy
sudo pip3 install image
sudo pip3 install matplotlib
sudo pip3 install torch
sudo pip3 install torchvision
```

- Run the test to check if it works

```bash
python3 test_simple.py --image_path assets/test_image.jpg --model_name mono+stereo_640x192
```

After that you should find two new files in the `asset` subfolder: A `test_image_disp.jpeg` which visualizes the depth-map and a `test_image_disp.npy` which is the Numpy depth-map itself.

### Monodepth2 installation on macOS Catalina

- Clone Monodepth2 repository

```bash
git clone https://github.com/nianticlabs/monodepth2.git
```

- Install Python3 and some common Python3 tools

```bash
brew install python3
```

- Install Python dependencies for Monodepth2

```bash
sudo pip3 install numpy
sudo pip3 install image
sudo pip3 install matplotlib
sudo pip3 install torch
sudo pip3 install torchvision
sudo pip3 install pillow==6.1
```

- Run the test to check if it works

```bash
python3 test_simple.py --image_path assets/test_image.jpg --model_name mono+stereo_640x192
```

After that you should find two new files in the `asset` subfolder: A `test_image_disp.jpeg` which visualizes the depth-map and a `test_image_disp.npy` which is the Numpy depth-map itself.