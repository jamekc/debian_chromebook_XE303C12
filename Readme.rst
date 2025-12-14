Installing Debian on an ARM Chromebook (XE303C12)
#################################################


:tags: debian, arm, chromebook
:authors: Pascal Geiser
:modifier: Jamekc
:summary: Debian installation on Samsung's ARM chromebook.

.. contents::

|ci-badge|

.. |ci-badge| image:: https://github.com/13pgeiser/debian_chromebook_XE303C12/actions/workflows/publish.yml/badge.svg
              :target: https://github.com/13pgeiser/debian_chromebook_XE303C12/actions/workflows/publish.yml/

Work heavily based on Kali ARM scripts: https://gitlab.com/kalilinux/build-scripts/kali-arm

Kernel config taken partially from: https://github.com/archlinuxarm/PKGBUILDs/tree/master/core/linux-armv7

I use the machine rarely. Do not expect much support if it does not fit your needs.

**Use at your own risk!**

Introduction
************

Chromebooks and ChromeOS are giving an excellent browsing experience. The system
is fast and with an ARM Chromebook the battery life is incredible.

The main issue with ChromeOS is the lack of good applications. All the google services
are perfectly integrated but apart from this, the overall application library is poor.
This fact has been partially addressed by the newest versions of ChromeOS with support
of Android applications. Sadly, my old Chromebook is slowly reaching end of life (July 2018
according to google: https://support.google.com/chrome/a/answer/6220366?hl=en) and it never
got support for Android applications.

As the HW still works perfectly, I decided to try installing my favorite OS: `Debian! <https://www.debian.org/>`__
The main debian page looks old and did not provide too much information:
https://wiki.debian.org/InstallingDebianOn/Samsung/ARMChromebook

The best source of information is from the guys of `Kali Linux <https://www.kali.org/>`__. They provide
`ARM images <https://www.offensive-security.com/kali-linux-arm-images/>`__ for a lot of different systems.

The scripts used to generate these images are available on `Gitlab <https://gitlab.com/kalilinux/build-scripts/kali-arm>`__
They provide an excellent basis to prepare a debian image.

Developer Mode
**************

The first step to take control of the Chromebook is to switch to developer mode. This disable the kernel signature verification
and let U run any firmware on the machine with the drawback of having at each startup an annoying warning screen...

* First, turn off the machine
* Press ESC + Refesh(F5) and Power-on @ the same time.
* On the recovery screen appears, press CTRL-D
* Confirm the switch by pressing enter and wait a (long) while.

Once developer mode is enabled, it's possible to enable USB boot:

* Boot the machine (pressing CTRL-D right after power-on).
* Once chrome is ready, press CTRL-Alt-F2
* Log as `chronos` with no password.
* su - # Should give you root access.
* Enable usb boot with::	crossystem dev_boot_usb=1 dev_boot_signed_only=0

For more, look here:
 * Chrome documentation: https://chromium.googlesource.com/chromiumos/docs/+/HEAD/developer_mode.md
 * ArchArm linux: https://archlinuxarm.org/platforms/armv7/samsung/samsung-chromebook

USB stick preparation (on a Linux host)
***************************************

- Download the latest zip archive from the [releases page](https://github.com/13pgeiser/debian_chromebook_XE303C12/releases) and unpack it.
- Open a terminal in the extracted folder
- plug a USB key to hold the debian installation image
- run *./install.sh* and select the USB key in the list.

**BEWARE it will erase all data on the selected disk!**

**Make sure you've selected the right one! You've been warned!**

Boot on USB stick
*****************

Once the USB key is ready, plug it in the **black** usb connector (ie USB 2.0) of
the chromebook . Start the machine and press ctrl-u (assuming you've already configured the
developer mode). Wait for the system to boot.

user::	root

passwd::	toor

To install on the local emmc drive, run as root (from the USB key):

::

	./install.sh /dev/mmcblk0

**This will wipe out the entire disk. You've been warned! ;-)**

Once done, shutdown the machine with `poweroff`. Do not `reboot` or you will get a black screen (bug in the backpanel driver?).

Have fun!

Installing XFCE
***************

Start the machine and hit ctrl-d to boot on the emmc (or ctrl-u if you want to test on a USB key first) and log as root:

user: root

passwd: toor

Setup a network connection::

`nmtui`

Note: Before running the XFCE installation script, make sure the system clock is correct.
Set your timezone with: `timedatectl set-timezone America/Santo_Domingo`
It is also recommended to install `ntpsec` and enable NTP to keep the time synchronized:
Example: `apt-get install ntpsec` and then `timedatectl set-ntp true`

Run the provided XFCE installation script::

	./xfce_install.sh

Wait for the installation to finish and `poweroff` before jumping in your prefered desktop (with power-on and ctrl-d).

Post‑Install XFCE
*****************

After installing and booting into XFCE, you may want to extend the system with Pi‑Apps,
a graphical app store originally developed for Raspberry Pi OS but compatible with Debian.

To install Pi‑Apps:

1. Download the installer script::

	wget -qO- https://raw.githubusercontent.com/Botspot/pi-apps/master/install | bash

2. Once finished, Pi‑Apps will appear in your XFCE menu under Accessories.

Note 1: Pi‑Apps provides easy installation of popular software (VS Code, Arduino IDE, Minecraft Pi, etc.)
It is optional, but recommended if you want a simple way to add applications to your XFCE desktop.

Note 2: If you encounter a "source.list missing" error during Pi‑Apps installation,
open and modify /etc/apt/sources.list with nano::

	sudo nano /etc/apt/sources.list

Then create a basic sources.list file for Debian Trixie::

	deb     http://deb.debian.org/debian trixie main contrib non-free non-free-firmware
	deb-src http://deb.debian.org/debian trixie main contrib non-free non-free-firmware

	deb     http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware
	deb-src http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware

	deb     http://deb.debian.org/debian trixie-updates main contrib non-free non-free-firmware
	deb-src http://deb.debian.org/debian trixie-updates main contrib non-free non-free-firmware

Save the file in nano (Ctrl+O, Enter) and exit (Ctrl+X).
Finally, run `sudo apt-get update` before retrying the Pi‑Apps installer.


Kernel upgrade
**************

The same script can be used to update the kernel and the modules on the emmc drive.

- Download the zip archive and unpack it from the running debian installation
- Open a terminal in the depacked folder
- run *sudo ./install.sh*

Known issues
************

The final result is usable but far from production quality.

1. Currently the machine does not like the reboot much. This leads to a back screen -> shutdown and restart each time.
2. Change the password!!! ;-)
3. Plenty of other problems not described here.

Rebuilding locally
******************

The scripts have been prepared to work in docker. To rebuild:
 * Install docker for your distro
 * Clone the repository: *git clone https://github.com/13pgeiser/debian_chromebook_XE303C12.git*
 * Jump in the folder: *cd debian_chromebook_XE303C12*
 * Call make: *make* and wait a while depending on your machine...

alternatively, instead of calling *make*, you can call *./scripts/docker_build.sh trixie* or
*./scripts/docker_build.sh bookworm* to force a particular release of debian.

Have fun!

