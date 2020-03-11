In Android development, adding a phone for usb debugging sometimes results in a permissions error.

### Steps to reproduce
Enter in termial:
```sh
$ adb devices
```
You will receive this response:

```
List of devices attached
xxxxxxxx    no permissions (user in plugdev group; are your udev rules wrong?);
see [http://developer.android.com/tools/device.html]
```

### To fix
list all devices:
```sh
$ lsusb
Bus 001 Device 002: ID 8087:8000 Intel Corp. 
Bus
 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 8085:8004 Xiaomi Inc. Redmi Note 3 (ADB Interface).
```

You will find your phone manufacturer's entry. Lie Xiami in this case.
Take note of the two numbers with colon between them 
`(xxx:xxx)`

Wee need to create a u dev permission for the device. To do this create the permission file in the /etc/** folder as shown bellow:

```sh
$ sudo vi /etc/udev/rules.d/51-android.rules
```
Enter the following in the file and save

``SUBSYSTEM=="usb", ATTR{idVendor}=="8085", ATTR{idProduct}=="8004", MODE="0666", GROUP="plugdev"
``

Replace `8085` (idVendor) and  `8004` (idProduct) with the corresponding values for your device.

Reload the udev rules

```sh
$ sudo udevadm control --reload-rules
```

Try listing the devices again
```sh
$ adb devices
List of devices attached
ZF6222Q9D9  device
```