LibreELEC setup
===============

Hardware
--------

* [8GB Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/)
* [Raspberry Pi Micro HDMI to HDMI cable](https://www.raspberrypi.com/products/micro-hdmi-to-standard-hdmi-a-cable/)
* [Raspberry Pi 5 Active cooler](https://www.raspberrypi.com/products/active-cooler/)
* [Raspberry Pi bumper](https://www.raspberrypi.com/products/bumper/)
* [256GB SSD kit](https://www.raspberrypi.com/products/ssd-kit/)
* [Raspberry Pi 27W USB-C power supply](https://www.raspberrypi.com/products/27w-power-supply/)
* [UGREEN M.2 NVMe SSD enclosure adapter](https://www.ugreen.com/products/ugreen-m-2-nvme-ssd-enclosure-adapter)

Basic set up
------------

First install the bumper and active cooler.

Write LibreELEC to SSD
----------------------

Disconnect the SSD from the SSD hat, but it in the SSD enclosure and connect to your laptop.

LibreELEC have their own [USB-SD Creator](https://libreelec.tv/downloads/) which automatically picks up the latest LibreELEC images.

But I've seen the macOS and Linux versions being unavailable for different reasons as different times. And last time I used it on Windows, it only showed 9.x images for the board I was using at the time (Odroid C2) even though 11.x images were available.

So, it's probably easier to manually download the [latest image](https://libreelec.tv/downloads/raspberry/) for the Raspberry Pi 5 and install it with the normal [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

Plug in the SSD enclosure, ignore any warnings about it being an unreadable drive, start the Imager and:

* Select _Raspberry Pi 5_ as the device.
* Click _Choose OS_ and then _Use custom_ and select the LibreELEC image that you downloaded (in my case `LibreELEC-RPi5.aarch64-12.0.2.img.gz` - you don't need to uncompress it first).
* Click _Storage_ and select the SSD - the name may be something non-obvious like "SAMSUNG MZ9LQ256HBJD-00B Media" but it also shows the disk size, 256.1 GB, so you can be confident its not pointing at your laptop's SSD.
* Then click _Next_ and select _No_ when asked about customizations (or set up ssh and other things if you want via customizations - I don't know if these customizations even work with LibreELEC images).
* For whatever reason, on macOS, the writing step required a "make changes" authorization (along with the more obvious permission to write to the external device).

That's it - exit the Imager, unplug the SSD enclosure, remove the SSD and put it back in the hat.

Install the SSD hat
-------------------

I thought the female header was for extending the Pi's GPIO pins, such that they stuck out above the hat, and plugged it into the hat first.

But it seems to be just about connecting the Pi to the hat - so plug it onto the GPIO pins first and then attach the hat.

The rubber of the bumper makes a little hard to know when to stop screwing the the bottom screws of the spacers - stop early rather than late (or you'll rip out the threads of the cheap nylon parts).

Power up
--------

Plug in a USB keyboard and mouse, ethernet, connect to monitor or TV and connect the USB-C power supply. The fan blows like crazy initially.

My Pi came with bootloader version `30de0ba5` (2023/10/30) and it didn't know how to boot from an SSD so, in the end I also had to set up a normal SD card.

I did this with the Imager (and just selected the normal Raspberry Pi 64 bit OS rather than LibreELEC) - this is a _much_ larger image than the LibreELEC image and takes much longer to write (and the SD card is a bit slower to write to than the SSD).

Once I'd written the SSD card and plugged it into the Pi (SD card "pins" facing upward - there's no spring click on inserting the card), it booted up fine.

The Pi locale set up is very strange - you can't select language or timezone independent of country, so I selected Switzerland and Zurich as the timezone and "Swiss High German" as the language but then ticked the boxes for "Use English language" and "Use US keyboard". Very poor design.

I let it update its software - as this was the whole point - to make sure it installed the latest bootloader.

Note: it would probably have been possible to update the bootloader without going thru all this setup, see the Pi [bootloader documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#bootloader_update_stable).

Once I'd gone thru the setup and rebooted the Pi, I could see that part of the process had been to update the bootloader:

```
$ vcgencmd bootloader_version
2025/02/12 10:51:51
version f788aab...
```

And I could see the SSD drive:

```
$ lsblk -io KNAME,TYPE,SIZE,MODEL
KNAME   TYPE   SIZE MODEL
mmcblk0 disk  14.7G
...
...
nvme0n1 disk 238.5G SAMSUNG MZ9LQ256HBJD-00BVL
...
```

Then I select _Logout_ from the Pi menu (top left) and then _Shutdown_. I unplugged it, removed the SD card and plugged it back in.

This time it certainly tried to boot from the SSD but just failed repeatedly.

All I actually saw was the screen power up and go into power saving mode repeatedly - I didn't e.g. see the reassuring Linux boot messages with it hitting some obvious point of failure.

So, I got out the Imager again and this time wrote the LibreELEC to the SD card (with the intention of then cloning it to the SSD once it was up and running on the Pi).

Booting LibreELEC from the SD card worked fine and I went thru the initial basic setup and:

* Enabled ssh (for `root` but with my own password)
* Disabled samba (as all my content is on a separate network share).

Cloning to SSD
--------------

I had hoped to now clone the SD card to SSD as described by Jeff Geerling [here](https://www.jeffgeerling.com/blog/2023/nvme-ssd-boot-raspberry-pi-5).

But LibreELEC is so cut down that it doesn't have `apt` and doesn't even have `bash` - instead, it's using [BusyBox](https://busybox.net/).

So, back to writing the normal Raspberry Pi 64 bit OS to an SD card (this time I used a new SD card so I could switch back and forward if needed).

And will try: <https://forum.libreelec.tv/thread/29112-fresh-install-on-nvme-ssd/?postID=195782#post195782>

Many pages tell you to update the boot order to `0xf461` but with the latest OS version this was the default:

```
$ rpi-eeprom-config
[all]
BOOT_UART=1
POWER_OFF_ON_HALT=0
BOOT_ORDER=0xf416
```

The `f416` bit of magic is read right-to-left and the nibbles are read as instructions (defined [here](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#BOOT_ORDER)) and so it goes:

* Try `6` (NVMe)
* Try `1` (SD card)
* Try `4` (USB mass storage)
* Do `f` (restart and begin the cycle again)

It the boot order isn't `0xf416`, then just run `sudo rpi-eeprom-config --edit` and change the value and `ctrl-X` to exit and update the settings.

Some pages say that Raspberry Pi OS will automatically detect and mount the SSD as a drive - that wasn't the case for me.

So, I did:

```
$ mkdir /tmp/libreELEC
$ sudo mount /dev/nvme0n1p1 /tmp/libreELEC
$ cd /tmp/libreELEC
$ sudo vim config.txt
```

And added the line `dtparam=nvme` as the last line (the drive is mounted writeable, the `sudo` is needed as `config.txt` is owned by root).

Then _Logout_ / _Shutdown_, unplug, remove SD card, plug in again. This time it all worked, thanks to _chewitt_ and this [post](https://forum.libreelec.tv/thread/29112-fresh-install-on-nvme-ssd/?postID=195782#post195782) that contained the `dtparam=nvme` magic.

Setting up Kodi
---------------

Kodi is designed to be used with a remote control with an exit or back button. When using the UI with a mouse it's a little odd - click in the upper-left corner to return to the previous screen.

Go to the cog icon (upper left) and:

* Select _Interface_ / _Regional_ and change e.g. time to 24h format and timezone.

Attaching shares
----------------

Setting up SMB via `ssh` seems the easiest approach.

Steps:

```
$ ssh root@libreelec.local
# ping daedalus.local
# mkdir /storage/daedalus
# vi /storage/.config/system.d/storage-daedalus.mount
```

Add this content:

```
[Unit]
Description=cifs mount script
Requires=network-online.service
After=network-online.service
Before=kodi.service

[Mount]
What=//daedalus.local/Multimedia
Where=/storage/daedalus
Options=username=a...,password=k...-n...,rw,vers=2.1
Type=cifs

[Install]
WantedBy=multi-user.target
```

Update `username` and `password` appropriately.

Then:

```
# systemctl enable storage-daedalus.mount
# systemctl status storage-daedalus.mount
# systemctl start storage-daedalus.mount
# ls /storage/daedalus
```

Then in the Kodi UI, hover over _TV Shows_ and click _Enter files section_, right click on _TV Shows_, select _Edit_ and then _Browse_, select `/storage/daedalus/TV Shows` and the rest is obvious.

Just leave it to chug - I interupted it once during this process and it wasn't happy. It's quite slow and _appears_ to hang at points.

Oddly, _TV Shows_ is already there and you just have to edit and browse to change the path associated with it.

For movies, I had to click _Add videos..._ and then _Browse_ but from then on it's the same.

Do the same for movies (remember click upper-left to get back to the main screen).

To set the time zone, click _Settings_ (the cog icon that's upper-left), then _Interface_ and _Regional_.

To change the view from just a list to something nicer, click left (or select _Options_ bottom-left) and change _Viewtype_ to _InfoWall_ (this includes a description on the left-hand side of the currently selected item) or _Wall_ (if you don't want the description).

I couldn't find any way to remove items or correct them without a mouse. With the mouse:

* Right click, select _Manager_ and remove unwanted items.
* Right click, select _Information_ and select _Refresh_ to find metadata, giving you the option to select between mutliple potential matches or manually search.

The _Enter_ button to the right of the zero on my remote brings up the menu that allows one to turn off subtitles - and _Exit_ gets one back to the movie.

---

Note that later SMB failed to start at least once on reboot but this could be fixed by ssh-ing in and:

```
# systemctl restart storage-daedalus.mount
```

But over time, I just found it easier to make sure the NAS was awake and then reboot Kodi (via its onscreen UI) rather than resorting to ssh.


---

Long pressing OK on my remote to bring up the context menu (so I can e.g. mark things as watched) seems to be a general issue over CEC.

Pressing all the buttons on my remote showed that there was a red button that caused it to flip X and a blue one that caused it to flip to Y.

So, using the details at <https://kodi.wiki/view/Keymap> first:

* Look for how long pressing OK is setup - what's it bound _to_.
* See if you can change the red button press to be long press.
* Could you get blue to bring up audio settings (so I can flip subtitles and audio language track).

---

_forced subtitles_ is a term for those subtitles that come on when needed during a film that's in your language, it's for those bits where e.g. some of the characters start speaking in Russian.

Ideally, you just want _forced subtitles_ for English. And anything that isn't English, should always use subtitles.

But it seems confusing how this should be done, my impression is that if "Preferred Subtitle Language" is set to "User Interface Language" (or English if the UI language isn't English) then it should work this out, i.e. if the film is in English it should work it is doesn't need subtitles (and should only use _forced subtitles_ if they exist) and if it's not in English it should choose the preferred subtitle language, i.e. English.

Just confirm "Preferred Subtitle Language" is set. And maybe try setting it to English and seeing if that makes a difference.

For an explanation of the confusion around the term _forced subtitles_, see <https://www.reddit.com/r/kodi/comments/mujddh/automatic_subtitles_only_for_specific_content/>

---

Naming
------

Many of the files, that Plex picked up without issue, were either missed entirely or mistaken for something else.

I found the for TV Shows, the best solution was to find the show on [The Movie DB](https://www.themoviedb.org/) (which despite its name is also where Kodi pull its TV show listings). Then name the containing folder of the seasons exactly as shown on The Movie DB, including the year, e.g. as "Battlestar Galactica (2004)" and then name the season folders S01 etc. and the episodes S01E01.mkv etc and to create a folder called Specials for any specials and name those episodes S00E01.mkv (using the episode number for the particular special that's shown in The Movie DB).

For movies, I followed a similar approach, name the containing folder exactly as in The Movie DB, e.g. "Guy Ritchie's The Covenant (2023)" and then making sure the actual movie file was named similarly but with spaces replaced by `.` and colons, apostrophes and most other punctuation removed, e.g. Guy.Ritchies.The.Covenant.2023.mkv

Clean library
-------------

Go to _Settings / System_ and bottom-left, toggle _Standard_ to _Advanced_, then back out of _System_ and go to _Media_ and in the _Library_ tab, select _Clean library_.

Go back to _System_ and toggle _Advanced_ back to _Standard_.

Reboot the system. You're still left with some top-level TV show folders but clearly containing 0/0 episodes. I suspect you can delete these with long-OK.

This just cleans up library references to files that no longer exist - it does _not_ remove any of your underlying media files.
