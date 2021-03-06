# Install Software

### Get the Raspberry Pi working.

Before we can do anything we have to get our car's computer connected to the 
internet. The fastest way is to use the disk image created for donkey cars. 

The method for using a disk image to create a bootable SD card varies between
operating systems. These instructions are for Ubuntu but you can see more 
instructions [here](https://www.raspberrypi.org/documentation/installation/installing-images/).

1. Download [zipped disk image](https://www.dropbox.com/s/vb9wlju4aqx7i5o/donkey_2.img.zip?dl=0) (900MB). 
2. Unzip the disk image.
3. Plug your SD card into your computer.
4. Open the "Startup Disk Creator" application.
5. Select your source disk image as the one you unzipped earlier.
6. Select your SD card as the disk to use. 
7. Click "Make startup disk".


### Setup the Pi's WiFi for first boot

We can create a special file which will be used to login to wifi on first boot. More reading [here](https://raspberrypi.stackexchange.com/questions/10251/prepare-sd-card-for-wifi-on-headless-pi), but we will walk you through it. 

On Windows, with your memory card image burned and memory disc still inserted, you should see two drives, which are actually two partitions on the mem disc. One is labled __boot__. On Mac and Linux, you should also have access to the __boot__ partition of the mem disc. This is formated with the common FAT type and is where we will edit some files to help it find and log-on to your wifi on it's first boot.

* Start a text editor: `gedit` on Linux. Notepad on Windows. TextEdit on a Mac.
* Paste and edit this contents to match your wifi:
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid=”<your network name>”
    psk=”<your password>”
}

```

Replace `<your network name>` with the ID of your network. Leave the quotes. I've seen problems when the network name contained an apostrophe, like "Joe's iPhone".
Replace `<your password>` with your password, leaving it surrounded by quotes. 
If it bothers you to leave your password unencrypted, you may change the [contents later](https://unix.stackexchange.com/questions/278946/hiding-passwords-in-wpa-supplicant-conf-with-wpa-eap-and-mschap-v2) once you've gotten the pi to boot and log-in.

* Save this file to the root of __boot__ partition with the filename `wpa_supplicant.conf`. On first boot, this file will be moved to `/etc/wpa_supplicant/wpa_supplicant.conf` where it may be edited later.

##### Setup Pi's Hostname
We can also setup the hostname so that your Pi easier to find once on the network. If yours is the only Pi on the network, then you can find it with 

```
ping raspberrypi.local
```

once it's booted. If there are many other Pi's on the network, then this will have problems. If you are on a Linux machine, or are able to edit the UUID partition, then you can edit the `/etc/hostname` and `/etc/hosts` files now to make finding your pi on the network easier after boot. Edit those to replace `raspberrypi` with a name of your choosing. Use all lower case, no special characters, no hyphens, yes undersores `_`. 

```
sudo vi /media/userID/UUID/etc/hostname
sudo vi /media/userID/UUID/etc/hosts
```

Now you're SD card is ready. Eject it from your computer, put it in the Pi 
and plug in the Pi.


### Connecting to the Pi

If you followed the above instructions to add wifi access you're Pi should
now be connected to your wifi network. Now you need to find it's IP address
so you can connect to it via SSH. 

The easiest way (on Ubuntu) is to use the `findcar` donkey command. You can try `ping raspberrypi.local`. If you've modified the hostname, then you should try: `ping <your hostname>.local`. I've sometimes had this fail on a windows machine.

If you are having troubles locating your Pi on the network, you will want to plug in an HDMI monitor and USB keyboard into the Pi. Boot it. Login with username: __pi__, password: __raspberry__. Then try the command:

```
ifconfig
```

Look for a section called wlan0. If this has a valid ip V4 address, 4 numbers separated with a dot, then you can try that with your ssh command. If you don't see anything like that, then your wifi config might have a mistake. You can try to fix with

```
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

If you don't have a hdmi monitor and keyboard, you can plug-in the Pi with a CAT5 cable to a router with DHCP. If that router is on the same network as your PC, you can try:

```
ping raspberrypi.local
```

Hopefully, one of those methods worked and you are now ready to ssh into your Pi. On Mac and Linux, you can open Terminal. On Windows you can install [Putty](http://www.putty.org/) or [one of the alternatives](https://www.htpcbeginner.com/best-ssh-clients-windows-putty-alternatives/2/).

If you have a command prompt, you can try:

```
ssh pi@raspberrypi.local
```

or

```
ssh pi@<your pi ip address>
```

or via Putty - username: __pi__, password: __raspberry__ hostname:`<your pi ip address>`


If you are using the prebuilt image specified above, then your Pi is ready to go. You should see a d2 and donkey directory. 

> Note: For a short time there was a bug in manage.py that used different settings for the PWM channel for steering and throttle. Open manage.py ```nano ~/d2/manage.py``` and make sure that you see the lines:
>
> * steering_controller = dk.parts.PCA9685(1)
> * throttle_controller = dk.parts.PCA9685(0)
>
> The (1) and (0) for the parts arguments should match whichever channel you used to plug your servo/ESC leads in to your 9685 board. Usually this ranges from 0-15 and it numbered on the board.

----
Now let's setup things on your PC. Install varies depending on platform.


## Install donkeycar on Linux

```
virtualenv env -p python3
source env/bin/activate
git clone https://github.com/wroscoe/donkey donkeycar
pip install -e donkeycar
```

----

## Install donkeycar on Windows

* Install [miniconda Python 3.6 64 bit](https://conda.io/miniconda.html)

* Install [git 64 bit](https://git-scm.com/download/win)

* From the start menu start the Andconda Prompt.

* Change to a dir you would like to use as the head of your projects.

```
mkdir projects
cd projects
```

* Get the latest donkey from github.

```
git clone https://github.com/wroscoe/donkey
cd donkey
```

* Create the python anaconda environmment

```
conda env create -f envs\windows.yml
activate donkey
```

* Install donkey source and create your local working dir:

```
pip install -e .
donkey createcar --path ~/d2
```

> Note: After closing the Anaconda Prompt, when you open it again, you will need to 
> type ```activate donkey``` to re-enable the mappings to donkey specific 
> python libraries

----

## Install donkeycar on Mac

* Install [miniconda Python 3.6 64 bit](https://conda.io/miniconda.html)

* Install [git 64 bit](https://www.atlassian.com/git/tutorials/install-git)

* Start Terminal

* Change to a dir you would like to use as the head of your projects.

```
mkdir projects
cd projects
```

* Get the latest donkey from github.

```
git clone https://github.com/wroscoe/donkey
cd donkey
```

* Create the python anaconda environmment

```
conda env create -f envs/mac.yml
source activate donkey
```

* Install Tensorflow

```
pip install https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-1.3.0-py3-none-any.whl
```


* Install donkey source and create your local working dir:

```
pip install -e .
donkey createcar --path ~/d2
```

> Note: After closing the Terminal, when you open it again, you will need to 
> type ```source activate donkey``` to re-enable the mappings to donkey specific 
> python libraries

-------

### Install another fork of donkeycar

Occasionally you may want to run with changes from a separate fork of donkey. You may uninstall one and install another. That's fastest, but leaves you with only the forked version installed:

``` 
pip uninstall donkeycar
git clone --depth=1 https://github.com/<username>/donkey donkey_<username>
cd donkey_<username>
pip install -e .
```

To get back to stock donkey install:

```
pip uninstall donkeycar
git clone --depth=1 https://github.com/wroscoe/donkey donkey
cd donkey
pip install -e .
```
