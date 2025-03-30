LibreElec setup
===============

Using `dd` on MacOS did not work.

Using LibreElec USD-SD Creator on a Windows VM did (there was no MacOS version at the time).

Oddly, the Creator only showed 9.x images for the Odroid C2 even though 11.x images were available.

So, I manually downloaded the 11.x image from <https://libreelec.tv/downloads/amlogic/> and selected it in the Creator.

I needed a USB mouse to do the initial setup, I enabled `ssh` with the default username and password.

At the start of the setup process various notifications appeared saying things had failed to start - this didn't seem to affect anything later.

The UI is a little odd - clicking in the upper-left corner returns you to the main screen.

Setting up SMB via `ssh` seems the easiest approach. Note that later SMB failed to start at least once on reboot but this could be fixed with:

```
# systemctl restart storage-daedalus.mount
```

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
Options=username=<nas-username>,password=<nas-password>,rw,vers=2.1
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
