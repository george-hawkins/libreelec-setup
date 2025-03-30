LibreELEC/Kodi set up on a Raspberry Pi 5
=========================================

After disappointing results with an [Odroid-C2](https://wiki.odroid.com/odroid-c2/odroid-c2), I wanted a setup that I felt was least likely to suffer glitching in playback and most likely to able to handle [HEVC/H.265](https://en.wikipedia.org/wiki/High_Efficiency_Video_Coding) content while being relatively easy to set up. So, I went with an overspec'd Raspberry Pi 5 setup running [LibreELEC](https://libreelec.tv/).

Note: at various points below, I reference forum posts and comments from [Christian Hewitt](https://github.com/chewitt) - he's the project lead and platform maintainer for LibreELEC.

Hardware
--------

This is a somewhat overspec'd setup - using an SSD rather than running things off an SD card apparently gains little (see this [post](https://forum.libreelec.tv/thread/28798-installing-libreelec-on-ssd/?postID=193919#post193919) from Christian Hewitt) and you can almost certainly use a base 2GB Pi 5 model without any problem - Christian Hewitt says, in this [post](https://forum.libreelec.tv/thread/?postID=179727#post179727) from 2023, that 1GB should be enough. I just wanted a setup that could have no excuses for poor playback.

* [4GB Raspberry Pi 5](https://www.raspberrypi.com/products/raspberry-pi-5/)
* [Raspberry Pi Micro HDMI to HDMI cable](https://www.raspberrypi.com/products/micro-hdmi-to-standard-hdmi-a-cable/)
* [Raspberry Pi 5 Active cooler](https://www.raspberrypi.com/products/active-cooler/)
* [Raspberry Pi bumper](https://www.raspberrypi.com/products/bumper/)
* [256GB SSD kit](https://www.raspberrypi.com/products/ssd-kit/)
* [Raspberry Pi 27W USB-C power supply](https://www.raspberrypi.com/products/27w-power-supply/)
* [Raspberry Pi SD card](https://www.raspberrypi.com/products/sd-cards/)
* [USB card reader](https://www.ugreen.com/products/2-in-1-usb-c-otg-card-reader)

I bought Raspberry Pi parts where possible, e.g. the HDMI cable, as unlike generic parts they're presumably guaranteed to work with the Pi (and e.g. in the case of the HDMI cable, Raspberry Pi clearly state it supports the highest resolutions etc. that the Pi itself supports).

Initially, I wanted to avoid ever having to boot the Pi off an SD card. I wanted to write the LibreELEC image onto the SSD directly from my laptop using an M.2 NVMe SSD adapter (like this [one](https://www.ugreen.com/products/ugreen-m-2-nvme-ssd-enclosure-adapter) from UGREEN). In the end it turned out that my Pi had come with a [bootloader](https://en.wikipedia.org/wiki/Bootloader) version that was too old to boot directly off an SSD and the easiest way to resolve this was to install the latest Raspberry Pi OS on an SD card and boot the Pi off this. This automatically updated the bootloader version and also meant I could write the LibreELEC image directly to the Pi M.2 HAT and its SSD.

In the end, I think this approach is easier and cheaper than using an SSD adapter even if you have a Pi with an up-to-date bootloader (primarily because, it turns out, you still have to make a minor modification to the image on the SSD after having written it before you can boot directly from it and this is trivial to do from Raspberry Pi OS but would less easy from Windows or macOS). The only downside is that you need a wired keyboard and mouse that you can plug into the Pi. Raspberry Pi do sell a suitable [keyboard](https://www.raspberrypi.com/products/raspberry-pi-keyboard-and-hub/) (that can also act as a USB hub) and [mouse](https://www.raspberrypi.com/products/raspberry-pi-mouse/) but there are cheaper options like the Logitech [MK120 keyboard and mouse bundle](https://www.logitech.com/en-us/shop/p/mk120-usb-keyboard-mouse.920-002565) for US$18.

The 32GB Raspberry Pi SD card is actually one of the cheapest class A2 microSD cards out there. It comes with a recent Raspberry Pi 32-bit OS image already installed (it's 32-bit so it can work with both older 32-bit Pis and the newer 64-bit ones). So, it should be possible to boot your Pi straight off this SD card and get everything up-to-date without the need for a USB card reader or the need to use your laptop/PC to write an image to the card. I actually used a generic microSD card and wrote the latest Raspberry Pi 64-bit OS image to the card from my laptop.

Note: technically if may be possible to get Raspberry Pi OS up and going using a wireless keyboard and mouse or setting things up such that you can do everything via `ssh` but this is more trouble than it's worth in my opinion.

Basic set up
------------

First install the bumper and active cooler (plugging the fan's connector into the corresponding connector on the Pi).

### Install the SSD hat

I thought the [female header](https://www.adafruit.com/product/4079), that comes with the SSD kit, was for extending the Pi's GPIO pins, such that they stuck out above the hat, and plugged it all the way into the hat first.

But the header is just for connecting the Pi to the hat - so plug it onto the GPIO pins first and then attach the hat.

The rubber of the bumper makes a little hard to know when to stop screwing the the bottom screws of the spacers - stop early rather than late (or you'll rip out the threads of the cheap nylon parts).

### Set up the SD card

Note: the initial point of booting off an SD card is to get the Pi's bootloader up-to-date. It is possible to do this by writing a tiny image solely designed to do to an SD card and then booting the Pi off this card (as described [here](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#bootloader_update_stable) in the section on using the Raspberry Pi Imager to update the bootloader). However, we're also going to use Raspberry Pi OS to write the LibreELEC image to the SSD so, we're going to write a full Raspberry Pi OS image to the SD card.

My Pi came with bootloader version `30de0ba5` (2023/10/30) which was created before Raspberry Pi released their SSD hat and so it didn't know how to boot directly from the SSD. But this was resolved by the following process.

I used the [Raspberry Pi Imager](https://www.raspberrypi.com/software/). Plug the microSD card into the card reader and plug it into your laptop, ignore any warnings about it being an unreadable drive, start the Imager and:

* Select _Raspberry Pi 5_ as the device.
* Click _Choose OS_ and select _Raspberry Pi OS (64-bit)_.
* Click _Storage_ and select the SSD - the name may be something non-obvious like "Generic STORAGE DEVICE  Media" but it also shows the disk size, e.g. 31.7 GB, so you can be confident its not pointing at your laptop's SSD.
* Then click _Next_ and select _No_ when asked about customizations (or set up `ssh` and other things if you want via customizations).
* For whatever reason, on macOS, the writing step required a "make changes" authorization (along with the more obvious permission to write to the external device).

The full Raspberry Pi OS image is quite large and takes a relatively long amount of time to write to the SD card (writing LibreELEC to the SSD later will be lightening fast in comparison and this really is more about image size rather than SD card vs SSD speeds).

Once written, unplug the SD card (the Imager automatically ejects it at the end of the write process so it's safe to remove). Plug the card into the Pi (with the card's "pins" facing upward, there's no spring click on inserting the card).

### Power up

After inserting the SD card, plug in everything else:

* Plug in a USB keyboard and mouse.
* Plug in ethernet.
* Connect to a monitor or TV.
* Connect the USB-C power supply.

The fan blows like crazy initially but quickly turns off.

Note: the Pi had Wi-Fi support but I chose to use wired ethernet.

The Pi booted fine and went straight into the initial OS setup wizard. Rather quaintly it defaults to _UK English_ as the default [locale](https://en.wikipedia.org/wiki/Locale_(computer_software)). The Pi locale setup is a little strange - you can't select language or timezone independent of country, so I selected Switzerland and Zurich as the timezone and "Swiss High German" as the language but then ticked the boxes for "Use English language" and "Use US keyboard". This assumption about what language you'll want to use based on your country seems odd in a product like the Pi. But one could just stick with the default choices as the intention is just to use the Raspberry Pi OS to get to a point where LibreELEC on the SSD can be used instead.

I let it complete the installation process, including letting it update all its software (to make sure it installed the latest bootloader).

At the end of the process, the Pi is rebooted.

### Checking the setup

Once the Pi had rebooted, I could see that it had automatically updated the bootloader without me having to do anything:

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

### Writing LibreELEC to the SSD

Previously, the _Raspberry Pi Imager_ was used on a laptop/PC to write the Raspberry Pi OS image to the SD card. Now, we're going to run it on the Raspberry Pi to write the LibreELEC image to the SSD.

Open _Chromium_ (under the _Pi_ menu - upper left - in the _Internet_ section) and go to the LibreELEC [Raspberry Pi downloads](https://libreelec.tv/downloads/raspberry/) and download the _Latest Stable Version_ for _Raspberry Pi 5_.

Now open the _Raspberry Pi Imager_ (under the _Pi_ menu in the _Accessories_ section).

Oddly, at the time of writing (late March, 2025), the _Imager_ looks visually the same on Raspberry Pi OS but has quite different options for the _Device_ and _Operating System_ fields.

In the _Imager_:

* Select _[ALL]_ as the device.
* Click _Choose OS_ and then _Use custom_ and select the LibreELEC image that you downloaded (in my case `LibreELEC-RPi5.aarch64-12.0.2.img.gz` - you don't need to uncompress it first).
* Click _Storage_ and select the SSD - the name may be something non-obvious like "SAMSUNG MZ9LQ256HBJD-00BVL". If you want, you can double check this matches the NVMe name seen when using `lsblk` but as before the _Imager_ is smart enough not to include your current system disk as one of the options.
* Then click _Next_ and select _No_ when asked about customizations (I don't know if the customizations even work with LibreELEC images).
* It then requires authentication to write to the SSD. And then, slightly strangely, to eject the drive - in the case of the SSD, this step actually does nothing.

That's it.

Some pages suggest you change the boot order so the bootloader tries the SSD drive first. But the default setting, where it tries the SD card first, is fine - once you remove the SD card, the cost of first checking the SD card before trying the SSD is negligible and also makes things easier if for some reason you later need to boot the Pi into Raspberry Pi OS or some other OS on an SD card.

If you really want to change the boot order, open a terminal, and enter `sudo raspi-config` and go to _Advanced Options_ and then _Boot Order_ and select the boot order you want.

### Failing to start LibreELEC

At this point, powered things down, removed the SD card and assumed that on being reconnected to power it would boot off the SSD and into LibreELEC.

It did **not**.

It certainly tried to boot from the SSD but just failed repeatedly. All I actually saw was the screen power up and go into power saving mode repeatedly - I didn't e.g. see the reassuring Linux boot messages with it hitting some obvious point of failure.

### Updating the LibreELEC image

It turns out you need to complete one more step while still running Raspberry Pi OS.

The LibreELEC image on the SSD expects to run from an SD card and a small modification is needed before it can successfully boot from an SSD.

You need to mount the SSD. Some pages say that Raspberry Pi OS will automatically detect and mount the SSD as a drive - that wasn't the case for me. It is possible to mount it using the _File Manager_ app but I did it in a terminal, and updated the necessary configuration, like so:

```
$ mkdir /tmp/libreELEC
$ sudo mount /dev/nvme0n1p1 /tmp/libreELEC
$ cd /tmp/libreELEC
$ sudo vi config.txt
```

Add the line `dtparam=nvme` as the last line in the file.

When I tried this first, `vi` told me the file was read-only when I tried to save it and I thought that I must have mounted the whole SSD drive read-only but `config.txt` is just read-only for most users as it's owned by root. Hence the need for `sudo`.

Then select _Logout_ from the Pi menu and then _Shutdown_. Unplug the Pi and remove the SD card.

### Successfully starting LibreELEC

Reconnect the Pi to power. This time it all worked, thanks to Christian Hewitt and this [post](https://forum.libreelec.tv/thread/?postID=195782#post195782) that contained the `dtparam=nvme` magic.

Initially, LibreELEC does some low-level setup and then reboots itself into the LibreELEC setup wizard. I went thru this and:

* On the _Networking_ page, there was nothing to do (or even select) as I was using wired ethernet and this was shown as being in "online" state.
* On the _Sharing and Remote Access_ page, I disabled Samba (as all my content is on a separate network share) and enabled `ssh` (for `root` but selecting the option to set my own password).

That's it - you're now running Kodi under LibreELEC.

Setting up Kodi
---------------

Note: Kodi is designed to be used with a remote control with an exit or back button but when using the UI with a mouse it's a little odd - click in the upper-left corner of the screen to achive the same as pressing your remote's back button and return to the previous screen.

The first thing, I did, was go to the cog icon (upper left) and selected * Select _Interface / Regional_ and:

* Changed _Region default format_ to _USA (24h)_
* Changed _Timezone country_ and _Timezone_ to match my location.

### Attaching shares

Setting up SMB via `ssh` from my laptop seemed like the easiest approach. My NAS advertises itself as `daedalus.local` on my local network so, you'll have to change any reference to `daedalus` to match your setup.

First `ssh` from your laptop to the LibreELEC device, using `ping`, check that it can see the NAS, make a mount-point for the NAS and create a SystemD mount configuration for it:

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
Options=username=<nas-username>,password=<nas-password>,rw,vers=2.1
Type=cifs

[Install]
WantedBy=multi-user.target
```

Note: this is a copy of the existing file `/storage/.config/system.d/cifs.mount.sample` that's been updated with the details of my NAS. When doing this you should check `cifs.mount.sample` to see if they've updated any of the recommended settings in the meantime.

Change `nas-username` and `nas-password` to match your setup, similarly `daedalus.local` and `Multimedia` need to be changed to match your seteup, `Multimedia` is the name of the share on my NAS that contains all my content.

Note: my NAS only supports an ancient version of SMB, hence the `vers=2.1` above. If you've got a newer NAS, you may be able to leave out the `vers` bit and let it select a more up-to-date version (search for "client max protocol" on the `smb.conf` [documentation page](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html) to see the latest supported version, it was 3.11 at the time of writing).

Now, enable and start things (the `start` step can take several seconds) and confirm you see the expected files under the mount-point:

```
# systemctl enable storage-daedalus.mount
# systemctl status storage-daedalus.mount
# systemctl start storage-daedalus.mount
# ls /storage/daedalus
```

### Setting content locations in Kodi

Now, the share is set up, head back to the Kodi UI. Hover over _TV Shows_ and, in the main area (to the right):

* Click _Enter files section_.
* **Right** click on _TV Shows_, select _Edit source_.
* Then _Browse_, select `/storage/daedalus/TV Shows` and click _OK_.
* Then a new dialog pops up, click _This directory contains_ and change its value from _None_ to _TV shows_ and click _OK_.
* Then click _Yes_ to "refreshing information for all items within this path."

Leave it to chug (you'll see a progress indicator upper-right that repeatedly ticks up to 100% for each show in turn). I interupted it once during this process and it wasn't happy. It seems to speed through some shows and for others, it _appears_ to hang at points (stuck as a given percentage for a minute or more). Just leave it be and do something else in the meantime. For my collection of about 70 shows (with an average of less than 20 episodes each), it took about 20 minutes to complete the refreshing step.

Now, go back to the main screen (click repeatedly in the upper-left corner) and do the same for _Movies_. Oddly, when you click on _Enter files section_, you see that entries already exist for "TV Shows" and "Videos" (which is another category of things, i.e. "videos" is not another word for "movies"). So, this time click "Add videos..." (perhaps a confusing name given what I said in the previous sentence) and:

* Click _Browser_, select `/storage/daedalus/Movies` (it'll fill in "Movies" as the name for the media source) and click _OK_.
* This time choose _Movies_ for the _This directory contains_ option. Leave the provider fields etc. unchanged and click _OK_.
* And, as before, click _Yes_ to "refreshing information for all items within this path."

As before, leave it to chug (it'll seem to hang every so often in the same manner at with TV shows). Unlike TV shows, where it just seems to go thru things in a random order, it does at least go thru things in alphabetical order so you have more idea of the progress it's making. For my collection of about 300 movies, this took about 15 minutes.

When you add more content later to your NAS and get Kodi to refresh its metadata for your shows and movies, it won't take anywhere near as long to do so.

Once I'd set up TV shows and movies, I went back to the main page and selected _Remove this main menu item_ for the sections:

* Music
* TV
* Radio
* Games
* Favorites
* Weather

For whatever reason, the sections _Pictures_ and _Videos_ don't seem to be removeable in this fashio so, I just left them.

### Updating how things look

Now, if you click on _Movies_, you'll just see a rather boring list. To change the view to something nicer, select _Options_ (bottom-left), or drag your mouse off the left hand edge of the screen, and change _Viewtype_ to _InfoWall_ (this includes a description on the left-hand side of the currently selected item) or _Wall_ (if you don't want the description).

Do the same in the _TV Shows_ section. For _TV Shows_, you also need to do this on the season and episode levels. Go into a series that has more than one season and change the _Viewtype_ setting for the seasons to _InfoWall_, then go into one on of the season and change the _Viewtype_ setting for the individual episodes to _InfoWall_ as well. Once you've changed it for one series, it's changed for all.

Managing your content
---------------------

On doing the initial refresh of metadata for my TV shows and movies, it was clear it had made a few mistakes. Previously, I'd used [Plex](https://www.plex.tv/) and on the whole it made a better job of determining what was what. It takes a little more care with naming files to get Kodi not to miss them altogether or to mistake a given show or movie for something else.

When Kodi has misrecognised something, it's oddly difficult to get it to show the underlying filename - right click on the item, select _Information_, then at the bottom of the screen, you should see a sequence of boxes, e.g. _Play_, _Your rating_ etc., and a blue right-pointing arrow. You don't seem to be able to click this arrow, the only way I found was to hover over the box to its left and scroll back and forward with the mouse's scroll wheel. Once you scroll all the way to the right, there's a _Refresh_ box - hover over it and the file location is shown below it. Yes, really - that seems to be it (see this forum [answer](https://forum.kodi.tv/showthread.php?tid=307864&pid=2804837#pid2804837)).

I found that for TV Shows, the best solution was to find the show on [The Movie DB](https://www.themoviedb.org/) (which despite its name is also where Kodi pull its TV show listings). Then name the containing folder of the seasons exactly as shown on The Movie DB, including the year, e.g. as "Battlestar Galactica (2004)" and then name the season folders S01 etc. and the episodes S01E01.mkv etc and to create a folder called Specials for any specials and name those episodes S00E01.mkv (using the episode number for the particular special that's shown in The Movie DB).

The big hint that helps Kody distinguish between two similarly named things is including the year in the name.

I found a situation where despite having the year there, incorrect punctuation caused Kodi to choose a film called "The Count of Monte-Cristo" rather than the more recent "The Count of Monte Cristo" - notice the hyphen in the first one, replacing the `-` with a `.` resolved the issue.

For movies, I followed a similar approach, name the containing folder exactly as in The Movie DB, e.g. "Guy Ritchie's The Covenant (2023)" and then making sure the actual movie file was named similarly but with spaces replaced by `.` and colons, apostrophes and most other punctuation removed, e.g. `Guy.Ritchies.The.Covenant.2023.mkv`.

Once you've change the names of files in this way, go to _Options_ and select _Update library_.

E.g. if you'd had a folder called "Andor" and Kodi had incorrectly recognized this as [_Andorra Between Two Evils_](https://www.themoviedb.org/tv/77109-andorra-entre-el-torb-i-la-gestapo) and you'd renamed it to "Star Wars: Andor (2022)" to match the naming in The Movie DB, you'll find that after the update, you can hopefully find your content now recongnised as expected _but_ Kodi will also have retained the incorrect entry.

Removing old incorrect entries is a bit problematic via your remote control (see later) but when using a mouse, just right click on the unwanted item, select _Manage_ and choose _Remove from library_.

Notes:

* There are ways to tell Kodi via its UI that its recognized some content incorrectly and to find the correct metadata but I found it sometimes lost these corrections when doing a subsequent update so, getting the naming of the underlying files correct seems the best way to do things.
* Some of my movies and shows come with subfolders of extras, e.g. interviews with the director etc. If these weren't specials that are covered in The Movie DB and which I could then name correctly, it could happen that Kodi would find these files and misinterpret them as something random. As I've never had any interest in tese kind of extras, I just removed them altogether but an alternative is to this just create a file called `.nomedia` in any folder that you want Kodi to ignore.

4K vs 1080p
-----------

The Pi can handle 4K but I found that when I had the setup connected to my 4K monitor, I noticed [tearing](https://en.wikipedia.org/wiki/Screen_tearing) when e.g. scrolling through movies in Kodi that I did not notice when the setup was connected to my 1080p TV.

Using the remote control
------------------------

Once connected to my TV, I could select Kodi from its usual input source menu (I usuaully have to select its refresh option before it detects Kodi if the Kodi device was turned on after the TV.

**Update:** actually, the TV detected the Pi once via searching for Anynet devices (my Samsung TV's term for CEC devices) and never found it again. This had worked with my older Odroid-C2 setup (which had lots of issues initial when it came to find it but without any obvious changes from me, this sorted itself out after a while). At the moment I'm using the setup with a wired mouse - this is a bit of pain tho' e.g. if you want to pause something vs just clicking pause. There seem to be no end of threads on about CEC not working with various TVs (particularly with gen 4 and 5 Pis).

TODO: I went into system / input / pheripheral settings, clicked CEC and change the CEC port to match the number of the physical one it was plugged into - 3 in my case - maybe it'll work on reboot.

Everything worked with my remote control much as you'd expect. The only thing I had to really discover was that the _Enter_ button to the right of the zero on my remote brings up the main play-time menu that allows one to e.g. turn off subtitles and the remote's _Exit_ button gets one back to the movie or TV show (pressing _Enter_ again pauses the movie rather than exiting, which seems a little unintuitive to me).

CEC alternatives
----------------

When CEC works it's great. But it seems to often be a nightmare and just not work with certain TVs.

Alternatives are:

* [Flirc USB](https://flirc.tv/products/flirc-usb-receiver) - a USB IR receiver that you can plug into your Pi and then use with nearly any TV remote. See also the Kodi Flirc [wiki page](https://kodi.wiki/view/FLIRC). In the US, it's available directly from the Flirc.tv, in other countries, you can get it [here](https://thepihut.com/products/flirc-usb-dongle-for-the-raspberry-pi) from the Pi Hut or from one of Flirc's [distributors](https://flirc.com/distributors).
* [OSMC remote control](https://osmc.tv/store/product/osmc-remote-control/) - if you're OK with using a second remote then the OSMC remote is designed for use with Kodi and comes with its own RF (rather than IR) dongle.

### Flirc setup

I've ordered a Flirc - if you go to its [product page](https://flirc.tv/products/flirc-usb-receiver) and then to the _Downloads_ section, there are installers macOS, Windows and Linux. My understanding is that you can, e.g. plug it into your laptop, use the installer suitable for your laptop, then get it paired and configured with your remote control and then swap it over to your Pi so avoiding having to do any setup there.

TODO
----

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

Power button
------------

Unlike earlier Pis, the Pi 5 has a power button to turn it on and off without having to unplug it.

For full details, see the [documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#power-button).

In summary:

* Click once to power up.
* Click once to bring up the shutdown/reboot/logout dialog.
* Click twice in quick succession to shutdown without brining up a dialog.
* Click and hold to hard shutdown.

The single and double click options obviously require the co-operation of the OS and the behavior described applies for Raspberry Pi OS. LibreELEC may behave differently - single click probably goes straight to shutdown.

TODO: the power button is a little fiddly to get, I'll probably end up soldering 2-pins of male header to the J2 jumper shown [here](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#add-your-own-power-button). I wish they'd provided with header already soldered on.

Note: initially, I thought the J2 connector was the same as the GPIO 20 pin plus ground that could be used on earlier Pis to signal to OS that it should shutdown. While it behaves similarly for shutdown, the GPIO 20 pin cannot be used to power up the Pi whereas the J2 connector can (see this Reddit [discussion](https://www.reddit.com/r/cyberDeck/comments/1ao9q23/) and this OpenWRT [post](https://forum.openwrt.org/t/j2-jumper-on-raspberry-pi-5/201158/16) for confirmation of the difference). If the GPIO 20 pin had been equivalent I would have bought some taller female header (like [this](https://thepihut.com/products/stacking-header-for-pi-a-b-pi-2-pi-3-2x20-extra-tall-header)) and connected a switch to it rather than soldering header to the J2 connector.

Clean library
-------------

Go to _Settings / System_ and bottom-left, toggle _Standard_ to _Advanced_, then back out of _System_ and go to _Media_ and in the _Library_ tab, select _Clean library_.

This just cleans up library references to files that no longer exist - it does _not_ remove any of your underlying media files.

Go back to _System_ and toggle _Advanced_ back to _Standard_.

Reboot the system. You're still left with some top-level TV show folders but clearly containing 0/0 episodes. I suspect you can delete these with long-OK.

Remounting the NAS
------------------

Every so often, I found LibreELEC would start without mounting the NAS (and so would still show all my content but report that the underlying files were missing if I tried to play anything). This could be fixed by ssh-ing in and:

```
# systemctl restart storage-daedalus.mount
```

But over time, I just found it easier to make sure the NAS was awake and then reboot LibreELEC/Kodi (via the onscreen UI) rather than resorting to ssh.

LibreELEC USB-SD Creator
------------------------

LibreELEC have their own [USB-SD Creator](https://libreelec.tv/downloads/) which automatically picks up the latest LibreELEC images.

But I've seen the macOS and Linux versions being unavailable for different reasons as different times. And last time I used it on Windows, it only showed 9.x images for the board I was using at the time (Odroid C2) even though 11.x images were available.

So, I just used the more manual approach above with the standard _Raspberry Pi Imager_.

Setting the boot order
----------------------

Many pages have instructions on how to set the boot order via the `BOOT_ORDER` EEPROM setting. But now you can do this much more conveniently using `raspi-config` as described above.

But for reference, here's how I did it this way once...

You can see the current EEPROM configuration like so:

```
$ rpi-eeprom-config
[all]
BOOT_UART=1
POWER_OFF_ON_HALT=0
BOOT_ORDER=0xf416
```

The hex value `f416` for `BOOT_ORDER` is interpretted as a sequence of instructions with each nibble being an instruction (defined [here](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#BOOT_ORDER)). Reading them in the correct order is a little difficult as the byte sequence is read right-to-left but the nibble are read left-to-right, so `f416` becomes the instruction sequence `1`, `6`, `4`, `f` which is intrepeted as:

* Try the SD card (`1`).
* Try the NVMe drive (`6`).
* Try USB mass storage (`4`).
* Restart and begin the cycle again (`f`).

You can change the order by running `sudo rpi-eeprom-config --edit` and changing `0xf416` to your preferred sequence, then press `ctrl-X` to exit and update the settings. You need to reboot the Pi for the settings to take effect (and for them to be shown when you run `rpi-eeprom-config`).

History
-------

My first media player was an [Nvidia Shield](https://en.wikipedia.org/wiki/Nvidia_Shield_TV) running Plex. It was great but died eventually.

The second one was an [Odroid-C2](https://wiki.odroid.com/odroid-c2/odroid-c2) running [LibreELEC](https://libreelec.tv/).

Technically the Odroid-C2 should be entirely capable of playing [AVC/H.264](https://en.wikipedia.org/wiki/Advanced_Video_Coding) content but playback that was perfect on my laptop was often glitchy on the C2, playback occassionally stopped and sometimes the whole device would hang. And almost as annoyingly fast-forwarding almost never worked - while fast-forwarding the picture would stop moving after a while but the time-offset would continue ticking up (and it wasn't the case that if you stopped fast-forwarding, the picture eventually caught up - it just remained stuck). And finally, some [HEVC/H.265](https://en.wikipedia.org/wiki/High_Efficiency_Video_Coding) wouldn't replay smoothly at all (it just stuttered along with the audio playing fairly smoothly but with the picture getting stuck every so often before catching up again).

My third media player is the Pi 5 setup with SSD that's described here. As the Pi seems to be the most popular platform for Kodi and LibreELEC, I hoped this iteration, with an overspec'd Pi setup, would perform noticeably better than the Odroid-C2. And it does and can replay HEVC content without issue.

If you're interested in seeing notes on setting up the Odroid-C2 or on writing LibreELEC to the SSD using an SSD adapter then see the `git` history for this file.
