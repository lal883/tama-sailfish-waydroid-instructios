# Instructions to run Waydroid on Sony Xperia XZ2C (Tama platform) running Sailfish OS

Sailfish OS release: 4.2.0.21, aarch64. Thanks to rinigus for the stable [Tama Sailfish port](https://github.com/sailfishos-sony-tama/main).

Thanks to Erfan Abdi for the instructions, modifications and and help with debugging over Telegram Waydroid channel.

Waydroid repository: https://github.com/waydroid/waydroid

Instructions are valid for Sailfish OS release 4.2.0.21 for Sony Tama and Waydroid release 1.1.1.

### 1. Install gbinder-python (thanks to piggz)
  http://repo.merproject.org/obs/nemo:/devel:/hw:/pine:/dontbeevil/sailfish_4.1.0.24_aarch64/aarch64/python3-gbinder-python-1.0.0+git1-1.10.1.jolla.aarch64.rpm


### 2. Install python3-gobject, lxc, dnsmasq
  ```bash
  devel-su zypper ref
  devel-su zypper in python3-gobject lxc lxc-libs
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

### 4. Initialise waydroid (also refer to https://docs.waydro.id)
  cd to the cloned Waydroid directory and run the python script as root. By default, this downloads and extracts images to /var/lib/waydroid/images/.

  ```bash
  devel-su python3 waydroid.py init
  ```
  Images can also be downloaded to a custom path by "devel-su python3 waydroid.py init -i /custom/path"

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

### 6. Have to resize the downloaded "system.img" in step 4 and replace the file /system/etc/ld.config.29.txt with the one attached. 
  
  This will be updated in later Waydroid releases as informed by Erfan, and this step wouldn't be needed.
  "resize2fs" was not available in Sailfish OS when I tried, so I copied the system.img to PC and did this over there and copied back the modified image to phone.
  
  If there is an easier way to do it on the phone itself, do mention.
  ```bash
  # on a PC, cd to the directory where you copied the system.img from phone
  mkdir waydroidrootfs
  resize2fs system.img 2G
  sudo mount system.img waydroidrootfs
  sudo cp /path/to/downloaded/ld.config.29.txt ./waydroidrootfs/system/etc/ld.config.29.txt
  sudo umount rootfs
  rm -rf waydroidrootfs
  ```
  Copy back "system.img" to phone.
  Will be probably a good idea to very sha256sum of the image before copying between the phone and PC to verify that the file wasn't corrupted during copying.

### 7. Modify the file /vendor/etc/vintf/manifest.xml
  ```bash
  devel-su vi /vendor/etc/vintf/manifest.xml
  ```
  Goto line 310, change "android.hardware.vibrator" to "android.hardware.vibrator.dis"
  Reboot device.


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
