### Initial System Setup

1: These steps use fresh install of Raspbian Stretch Lite
2: Update and upgrade
    sudo apt-get update
    sudo apt-get dist-upgrade
3. Try the upgrade a second time

======================

NOTE: Possible pre install files: 
```
sudo apt install build-essentials bc raspberrypi-kernel-headers vim
```
You may replace vim with your text-editor.


Taken from second answer on https://raspberrypi.stackexchange.com/questions/64502/install-drivers-for-rtl8812au-wireless-usb-adapter

<br>

### Building 8812AU wifi drivers

========================================

1. Download wifi-install:
    ```
    sudo wget http://www.fars-robotics.net/install-wifi -O /usr/bin/install-wifi & sudo chmod +x /usr/bin/install-wifi
    ```
2. Run the program to check for driver if rpi-update is run.
    ```
    sudo install-wifi -c rpi-update
    ```
4. If a driver is available from step 3, run `rpi-update` to update firmware.
    ```
    sudo rpi-update
    ```
5. update the driver for the new kernel installed by `rpi-update`.
    ```
    sudo install-wifi -u rpi-update
    ```
6. reboot to update the kernel with the new wifi driver.
    ```
    sudo reboot
    ```
7. Login and verify driver
    ```
    modprobe 8812au
    ```

**NOTE**: 
You can try this other method if this doesn't work for you: https://www.max2play.com/en/forums/topic/howto-raspberry-pi-3-realtek-802-11ac-rtl8812au/
