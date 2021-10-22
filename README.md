## Note: Waydroid has been packaged for Sailfish OS in [Chum repo](https://github.com/sailfishos-chum/main). It is suggested to install it for there.



# Instructions to run Waydroid on Sony Xperia XZ2C (Tama platform) running Sailfish OS

Thanks to rinigus for the stable [Tama Sailfish port](https://github.com/sailfishos-sony-tama/main).

Thanks to Erfan Abdi for the instructions, modifications and help with debugging over Waydroid Telegram channel.

Waydroid repository: https://github.com/waydroid/waydroid

Instructions are valid for Sailfish OS release 4.2.0.21 for Sony Tama and Waydroid release 1.1.1.

### 1. Install gbinder-python (thanks to piggz and HengYeDev)
  For aarch64: http://repo.merproject.org/obs/nemo:/devel:/hw:/pine:/dontbeevil/sailfish_4.1.0.24_aarch64/aarch64/python3-gbinder-python-1.0.0+git1-1.10.1.jolla.aarch64.rpm
  
  For armv7hl: https://repo.sailfishos.org/obs/home:/heng/sailfish_latest_armv7hl/armv7hl/python3-gbinder-python-1.0.0+git1-1.1.1.jolla.armv7hl.rpm


### 2. Install python3-gobject, lxc, dnsmasq
  ```bash
  devel-su zypper ref
  devel-su zypper in python3-gobject lxc git
  ```

  Add [Chum repo](https://github.com/sailfishos-chum/main) and install dnsmasq from there.
  ```bash
  # add chum repo and
  devel-su zypper ref
  devel-su zypper in dnsmasq
  ```

### 3. Clone waydroid repo
  ```bash
  git clone https://github.com/waydroid/waydroid.git
  ```
  Note: there have been reports that cloning the repo directly under user home location removes the contents of cloned directory wehn running some Waydroid commands. It is suggested to clone the repo within a subfolder under user home until this gets updated.

### 4. Initialise waydroid (also refer to https://docs.waydro.id)
  cd to the cloned Waydroid directory and run the python script as root. By default, this downloads and extracts images to /var/lib/waydroid/images/.

  ```bash
  devel-su python3 waydroid.py init
  ```
  Images can also be downloaded to a custom path by "devel-su python3 waydroid.py init -i /custom/path"
  
  If the images were downloaded to the default location, these can be moved to different location (like home partition) manually and free up space in the root partition. Be sure to update the new path to image in /var/lib/waydroid/waydroid.cfg after the drill.

### 5. Create "/etc/gbinder.d/alien.conf" with the following content
  
  ```bash
  [Protocol]
  /dev/puddlejumper = aidl2
  /dev/vndpuddlejumper = aidl2
  /dev/hwpuddlejumper = hidl

  [ServiceManager]
  /dev/puddlejumper = aidl2
  /dev/vndpuddlejumper = aidl2
  /dev/hwpuddlejumper = hidl
  ```
  File attached.

### 6. Have to resize the downloaded "system.img" in step 4 and replace the file in the image with the one attached. 
  #### With the recent Waydroid images update this step is probably not needed anymore. Skip this step and if Waydroid fails to start revisit this.
  
  This will be updated in later Waydroid releases as informed by Erfan, and this step wouldn't be needed.
  "resize2fs" was not available in Sailfish OS when I tried, so I copied the system.img to PC and did this over there and copied back the modified image to phone. However, check while logged in as `root`, it should be at `/sbin/resize2fs`, part of `e2fsprogs`.
  
  ```bash
  # cd to the directory where you have the system.img on the phone or copied to a PC
  mkdir waydroidrootfs
  resize2fs system.img 2G
  sudo mount system.img waydroidrootfs
  sudo cp /path/to/downloaded/ld.config.29.txt ./waydroidrootfs/system/etc/ld.config.29.txt
  sudo umount waydroidrootfs
  rm -rf waydroidrootfs
  ```
  
  If the image was corrected on a PC, copy back the "system.img" to phone.
  Will be a good idea to verify sha256sum of the image before transferring between the phone and PC to be sure that the file wasn't corrupted during the transfer process.

### 7. Modify the file /vendor/etc/vintf/manifest.xml
  This step is optional and specifically applicable to Sony Tama devices. Other devices may be able to get Waydroid working without it.
  
  ```bash
  devel-su vi /vendor/etc/vintf/manifest.xml
  ```
  Goto line 310, change "android.hardware.vibrator" to "android.hardware.vibrator.dis"
  Reboot device.

  Note: The above procedure essentially disables vibration in Waydroid. Alternately, to enable vibration in Waydroid (thanks to @piggz) edit "/usr/libexec/droid-hybris/system/etc/init/disabled_services.rc" and remove the line "service vendor.vibrator-1-0 /vendor/bin/hw/android.hardware.vibrator@1.0-service_HYBRIS_DISABLED" and restart the device

### 8. Open a console window and start Waydroid container as super user
  (also refer to https://docs.waydro.id)
  waydroid.py script is in the directory from step 3
  ```bash
  # cd to the waydroid directory
  devel-su python3 waydroid.py container start
  ```

### 9. In another console window start Waydroid session as regular user and wait until you see a message "Android with user 0 is ready"
  ```bash
  python3 waydroid.py session start
  ```

### 10. Open a third console window and execute as regular user to get the Waydroid UI
  ```
  python3 waydroid.py show-full-ui
  ```

The last three steps could be simplified probably. Clicking Waydroid or android app icons in the app grid were not opening them at least now. Executing command in step 10 was, though.

### Launching Waydroid apps from Sailfish app grid
Thanks to Taepi on Waydroid Telegram channel.

Add a symlink to the python script.
```bash
devel-su ln -s full/path/to/waydroid.py /usr/local/bin/waydroid
```
Tapping the Waydroid app icon from Sailfish app grid should open the Waydroid app window now.

Since Waydroid multi-windows option wasn't working when I tried, couldn't open two Waydroid app windows simultaneously. Clicking a new app icon drew over the previouly open waydroid window.

Hint: with symlink in place, "python3 waydroid.py" in all previous steps way be replaced by simply "waydroid". However, you must still be in the directory where waydroid was cloned to to run some of the waydroid commands.

### Sailfish - Waydroid integration
Some level of integration can be achieved between Sailfish and Waydroid instance by making use of KDE Connect. This would allow sharing clipboard contents, file transfers and media control.

Install "Sailfish Connect" from OpenRepos.net on Sailfish and  install KDE Connect android application under Waydroid instance. Pair it and that is it. Ability to control media playing in Waydroid (eg. Spotify) from Sailfish Connect can be useful that the Waydroid UI could be completely closed from the Sailfish home screen yet have control over media playback.

Unfortunately the current implementation of Sailfish Connect do not have the plugin to receive notifications from a connected device yet. It would have been convinient if Waydroid notifications could be transferred to Sailfish OS through KDE connect.

## Found the following to be working/not working currently
* Camera (no flash and might crash with extended use)
* Spotify
* Whatsapp (audio calls work, haven't tested video calls)
* PrimeVideo
* Browser (video playback is inconsistent, sometimes it works and sometimes not. Had better luck with Firefox generally)
* KDE connect could be used for file and clipboard sharing between Sailfish and Waydroid
* No notification sounds, and notifications are restricted to Waydroid only
* No vibrations for/inside Waydroid events
* Apps couldn't access location
*  .. if you tried Waydroid, do mention more here, or add your fixes

## Other Tips
* Restart the container directly if it feels like Waydroid is misbehaving. Waydroid session should automatically start in a few seconds, so that Android appps can be directly opened from the grid ```devel-su waydroid container restart```
