# Raspberry Pi Webcam

## Goal
The goal of theese instructions is to provide a way for a Raspberry Pi (RPi) to act as a 'webcam' at CRF. The webcam images are then regularly published to the Image API in [crf-dashboard](https://github.com/ChalmersRobotics/crf-dashboard) using a [Node-RED](https://nodered.org/) flow on our local server. This replaces the code in the [crf-webcam](https://github.com/ChalmersRobotics/crf-webcam) repository, but does not include the same amount of features. Most importantly it lacks the annonymize feature which used to blur faces (but did not work very well) and the 'detect movement' feature.


This repository contains instructions for installing *mjpeg_streamer* on a RPi running Raspbian for making it act as a simple network attached camera, and how to keep the RPi updated automaticaly.

> Note: Theese instructions have been testen on a Raspberry Pi 2 Model B running Raspbian Stretch Lite. There is no guarantee that the method described here works without requiring any modifications on your system. 

## Installation

First, make sure that your Pi is updated
```bash
sudo apt-get update
sudo apt-get upgrade
```
and install the required dependencies
```bash
sudo apt-get install cmake libjpeg8-dev 
```

Clone [jacksonliam's fork](https://github.com/jacksonliam/mjpg-streamer) of [mjpg-streamer](https://sourceforge.net/projects/mjpg-streamer/) from GitHub:
```bash
git clone https://github.com/jacksonliam/mjpg-streamer.git
```

and then enter the code directory:
```bash
cd mjpg-streamer/mjpg-streamer-experimental
```

Next, build and install mjpg-streamer and its dependencies using
```bash
make
sudo make install
```

> The compilation should finnish without any errors.

You can now test your setup by running the following command (assuming that your camera is plugged in and working using `raspistill`)
```bash
mjpg_streamer -o "output_http.so" -i "input_raspicam.so --width 1640 --height 1232 -fps 15 -quality 10"
```
> Feel free to configure the options to your liking according to the [documentation](https://github.com/jacksonliam/mjpg-streamer/blob/master/mjpg-streamer-experimental/README.md).

The stream should now be accessible by visiting `http://<IP>:8080/?action=stream`. If you want snapshot jpeg images, visit `http://<IP>:8080/?action=snapshot` instead. Press Ctrl-C to stop the stream when you are done.

If you want the stream to start at boot, edit the file `/etc/rc.local` and add the above command, followed by an ampersand (`&`) right before the `exit 0` line. The line should look like:
```bash
mjpg_streamer -o "output_http.so" -i "input_raspicam.so --width 1640 --height 1232 -fps 15 -quality 10" &
```

Reboot to make sure it works:
```bash
sudo reboot
```

## Automatic update
When mounting several Raspberry Pi's (such as at CRF), one does not want to have to manually keep them updated. Here is where `crontab` comes into play. Using the following cron jobs, the RPi will automatically check for, and update, packages and also perform a system reboot.

Simply edit your *crontab* file using 
```bash
sudo crontab -e
```

and add the following lines to it
```bash
0 2 * * 0 sudo apt -y update
0 3 * * 0 sudo apt -y dist-upgrade
0 5 * * 0 sudo reboot
```
Theese lines simply make the RPi update every sunday starting with `apt update` at 02:00, `apt dist-upgrade` at 03:00 and lastly a reboot at 05:00. The `-y` flag make sure the commands run without any user interaction. See the [crontab quick refence](https://www.adminschoice.com/crontab-quick-reference) for a more in-depth explanation about the definition of cron jobs.


## Additional Links
For more information and/or troubleshooting, here are some links that may help.
* [jacksonliam/mjpg-streamer](https://github.com/jacksonliam/mjpg-streamer)
* [Secure Webcam streaming with MJPG-Streamer on a Raspberry Pi](https://www.sigmdel.ca/michel/ha/rpi/streaming_en.html)
* [Install mjpg-streamer on Raspberry Pi for video streaming!](https://www.collaborizm.com/thread/SyFenrp6l)
* [Five Ways To Run a Program On Your Raspberry Pi At Startup](https://www.dexterindustries.com/howto/run-a-program-on-your-raspberry-pi-at-startup/)
* [How to stream the picamera to your browser](https://desertbot.io/blog/how-to-stream-the-picamera)
* [Automatically update and upgrade Raspbian](https://hjerpbakk.com/blog/2018/06/18/automatically-update-and-upgrade-raspbian)