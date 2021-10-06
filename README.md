# Instructions for running Waydroid on Sony Xperia XZ2C

Sailfish OS release: 4.2.0.21, aarch64. Thanks to rinigus for the stable Tama Sailfish port.

Thanks to Erfan Abdi for the instructions, modifications and and help with debugging over Telegram Waydroid channel.

Waydroid repository: https://github.com/waydroid/waydroid

### 1. Install gbinder-python (thanks to piggz)
  http://repo.merproject.org/obs/nemo:/devel:/hw:/pine:/dontbeevil/sailfish_4.1.0.24_aarch64/aarch64/python3-gbinder-python-1.0.0+git1-1.10.1.jolla.aarch64.rpm


### 2. Install python3-gobject, lxc, dnsmasq
  ```bash
  devel-su zypper in python3-gobject lxc
  ```

Add chum repo and install dnsmasq from there.
[Chum repo](https://github.com/sailfishos-chum/main)

### 3. Clone waydroid repo
  ```bash
  git clone https://github.com/waydroid/waydroid.git
  ```

### 4. Initialise waydroid (also refer to https://docs.waydro.id)
  cd to the cloned Waydroid repo and run the python script as root. This downloads and extracts images to /var/lib/waydroid/images.

  ```bash
  devel-su python3 waydroid.py init
  ```
  Images can also be downloaded to a custom path by "devel-su python3 waydroid.py init -i /custom/path"

### 5. Create "/etc/gbinder.d/alien.conf" with the following content
  File attached.

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

### 6. Have to resize the downloaded "system.img" in step 4 and replace the file /system/etc/ld.config.29.txt with the attached one. 
  
  This will be updated in later Waydroid releases as informed by Erfan, and this step wouldn't be needed.
  "resize2fs" was not available in Sailfish OS when I tried, so I copied the system.img to PC and did this over there and copied back the modified image to phone.
  
  If there is an easier way, do mention.
  ```bash
  mkdir waydroidrootfs
  resize2fs system.img 2G
  sudo mount system.img waydroidrootfs
  sudo cp <updated ld.config.29.txt> rootfs/system/etc/ld.config.29.txt
  sudo umount rootfs
  rm -rf waydroidrootfs
  ```
  <copy back system.img to phone>

### 7. Edit the file /vendor/etc/vintf/manifest.xml
  ```bash
  devel-su vi /vendor/etc/vintf/manifest.xml
  ```
  Goto line 310, change "android.hardware.vibrator" to "android.hardware.vibrator.dis"
  Reboot device.


### 8. Open a console window and start Waydroid container as super user (also refer to https://docs.waydro.id)
  waydroid.py script is in directory cloned in step 3 
  ```bash
  devel-su python3 waydroid.py container start
  ```

### 9. In another console window start Waydroid session as regular user and wait until you see a message "Android with user 0 is ready"
  ```bash
  python3 waydroid.py session start
  ```

### 10. Open a third console window and execute to get the Waydroid UI
  ```
  python3 waydroid.py show-full-ui
  ```

The last three steps could be simplified I believe. Clicking Waydroid or android app icons in the app grid were not opening them at least now. Executing command in step 10 was, though.   
